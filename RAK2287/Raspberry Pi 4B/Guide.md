# Set up a DIY Miner & Packet Forwarder, using either a Pi or a cloud miner.

## Image your Raspberry Pi microsd card
+ Download the Raspberry Pi OS 64 bit lite image
` http://downloads.raspberrypi.org/raspios_lite_arm64/images/`

+ Flash it to your micro sd card using using a tool such as Etcher or Raspberry Pi imager.

+ Create new (empty) file on root of `boot` SD card called `ssh`
```touch ssh```

## Pi basic Config

+ SSH into raspberry pi

```console
ssh pi@IP-ADDRESS' where IP-ADDRESS is the local IP of the raspberry pi
```
The default password is `raspberry`

+ Change your password 
```console
passwd
```

+ Launch Raspbery Pi configurator
```console
sudo raspi-config
```
    - Select System Options
        - Select Hostname
        - Choose a new hostname [initials-location-MINER]
    - Select `Interfacing Options`
        - Select `SPI`
        - Select `Yes`
    - Select `Interfacing Options`
        - Select `I2C`
        - Select `Yes`
    - Select `Interfacing Options`
        - Select `Serial Port`
        - Select `No` for shell access
        - Select `Yes` for serial port hardware
        - Select `Yes` to confirm
  
       - Save changes and reboot by selecting `Finish`
       
+ Change the user to "miner" from "pi"

    ```console
    sudo adduser miner
    ```
    
+ Give user "miner" sudo permissions

    ```console
    sudo usermod -aG sudo miner
    ```
    
+ Reboot

    ```console
    sudo reboot
    ```

+ SSH back into raspberry pi as miner

+ Increase the swap file size
    - Turn off swap

    ```console
    sudo dphys-swapfile swapoff
    ```

    - Edit config file
    ```console
    sudo nano /etc/dphys-swapfile
    ```

    - Edit the line 'CONF_SWAPSIZE=100' to 'CONF_SWAPSIZE=1024"
    - Press CTRL-X, and then Y, and then Enter to save changes
    - Reboot with `sudo reboot`

+ Update software on the pi
    - Update & upgrade software repo cache
    ``` console
    sudo apt update && sudo apt dist-upgrade -y && sudo apt autoremove
    ```

    - Install git and jq 
    ```console
    sudo apt install git jq
    ```

+ Install Docker using convience script from https://docs.docker.com/engine/install/debian/

    - Download the script 
    ```console
    curl -fsSL https://get.docker.com -o get-docker.sh
    ```

    - Run the install script 
    ```console 
    sudo sh get-docker.sh
    ```

    - Remove the install script.
    ```console
    rm get-docker.sh
    ```

    - Allow miner user to run docker by adding miner to the docker group.
    ```console
    sudo usermod -aG docker miner
    ```
    
    - Reboot for changes to take affect.
    ```console
    sudo reboot
    ```

## Set up Miner on Pi
Source: https://developer.helium.com/blockchain/run-your-own-miner

+ SSH into raspberry pi (if not already connected).
```console
ssh miner@raspberrypi.local
```

+ Create directory for miner data
```console
mkdir ~/miner_data
```

+ Create directory for overlay
```console
mkdir ~/overlay
```

+ Set up how overlay will work

```console
cd overlay
```

+ Create the Sys Config file

```console
nano sys.config
```

+ Paste this into that config file

```console
%% -*- erlang -*-
[
    "config/sys.config",
    {lager, [
        {log_root, "/var/data/log"}
    ]},
    {blockchain, [
        {key, undefined}
    ]},
    {miner, [
        %% bind jsonrpc to host when in a container
        {jsonrpc_ip, {0, 0, 0, 0}},
        {use_ebus, false},
        {gateway_and_mux_enable, true},
        {gateways_run_chain, false},
        {radio_device, {{0, 0, 0, 0}, 1680, deprecated, deprecated}}
    ]}
].
```

- Press CTRL-X, and then Y, and then Enter to save changes   

+ Get the filename for the most recent version of the miner docker image from quay.io/team-helium/miner. Make sure to get the CORRECT (arm64 vs amd64) version for the pi. They are in the format `miner-xxxNN_YYYY.MM.DD` to the current one at the time of this document is `miner-arm64_2020.09.08.0_GA`.

+ This docker command will download docker image and set it to always start up. Be sure to swap out `miner-xxxNN_YYYY.MM.DD.0_GA` for the current image name. Also, if you're running the cloud use amd64, if on a Pi arm64

```console
docker run -d \
   --restart always \
   --publish 1680:1680/udp \
   --name miner \
   --mount type=bind,source=/home/miner/miner_data,target=/var/data \
   --mount type=bind,source=/home/miner/overlay/sys.config,target=/config/sys.config \
quay.io/team-helium/miner:miner-arm64_2021.11.11.0_GA

```

+ Verify that your container has started
```console
docker ps
```

+  Allow miner user to move files on and off (useful for moving the swarm key off)
    ```console
    sudo chown -R miner:miner ~/miner_data/miner/
    ```

## Set up Packet Forwarder

+ Make sure you're in your home directory
```console
cd ~
```

+ Clone the packet forwadrer for sx1302 based gateways like the RAK2287
```console
git clone https://github.com/helium/sx1302_hal.git
```

+ Move to the project folder 
```console
cd sx1302_hal
```

+ If you do not want to have to put in your password a million times in the next step, you can create a ssh key in the sub steps below otherwise skip to the next step.
    - Create the ssh key pair.

    ```console
    ssh-keygen -t rsa
    ```
    
    Press enter three times to accept the defaults.

    - Set up your user to use the key pair.

    ```console
    ssh-copy-id -i ~/.ssh/id_rsa.pub pi@localhost
    ```
    
    You will need to enter your password.

+ Compile the project
```console
make clean all
```

+ Make the executables 
```console
make install
```

Note you can change parameters for this step in `target.cfg`

+ Install the config files 
```console
make install_conf
```

+ Move into bin directory 
```console
cd bin
```

+ Make a copy of the conf file for your reggion and name it as the default
```console
cp global_conf.json.sx1250.US915 global_conf.json
```

+ Add pi user to gpio group
```console
sudo usermod -aG gpio pi
```

+ Reboot for changes to take effect 
```console
sudo reboot
```

+ SSH back in to pi
```console
ssh pi@raspberrypi.local
```

+ Move to lora packet folder bin directory
```console
cd sx1302_hal/bin
```

+ Test out the packet forwarder 
```console
./lora_pkt_fwd
```

+ Stop the packet forwarder by pressing CTRL-c


## Set up auto start scripts
Source: https://github.com/Lora-net/sx1302_hal/tree/master/tools/systemd

+ Create a systemd service script 
```console
sudo nano /etc/systemd/system/lora_pkt_fwd.service
```

+ Paste the following into new text file:

```
[Unit]
Description=LoRa Packet Forwarder
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/pi/sx1302_hal/bin
ExecStart=/home/pi/sx1302_hal/bin/lora_pkt_fwd -c /home/pi/sx1302_hal/bin/global_conf.json
Restart=always
RestartSec=30
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=lora_pkt_fwd
User=pi

[Install]
WantedBy=multi-user.target
```

+ Save the file by pressing CTRL-X, and then Y, and then Enter to save changes

+ Reload the daemon 

```console
sudo systemctl daemon-reload
```

+ Enable the service 

```console
sudo systemctl enable lora_pkt_fwd.service
```

### The following commands to disable the service, manually start/stop it and check status:

```console
sudo systemctl disable lora_pkt_fwd.service
```

```console
sudo systemctl start lora_pkt_fwd.service
```

```console
sudo systemctl stop lora_pkt_fwd.service
```

```console
sudo systemctl status lora_pkt_fwd.service
```

### Configure rsyslog to redirect the packet forwarder logs into a dedicated file

```console
sudo nano /etc/rsyslog.d/lora_pkt_fwd.conf
```

+ Paste the following into the text editor:
```
if $programname == 'lora_pkt_fwd' then /var/log/lora_pkt_fwd.log
if $programname == 'lora_pkt_fwd' then ~
```

+ Save the file by pressing CTRL-X, and then Y, and then Enter to save changes

+ Restart the rsyslog service

```console
sudo systemctl restart rsyslog
```

+ Reboot for changes to take affect.

```console
sudo reboot
```

### See the Lora Packet Forwarder logs

```console
tail /var/log/lora_pkt_fwd.log
```
## Optional - Set up Miner autoupdate script
Source: https://github.com/Wheaties466/helium_miner_scripts

+ Clone miner auto update script repo
```console
git clone https://github.com/Wheaties466/helium_miner_scripts.git
```

+ CD into directory
```console
cd helium_miner_scripts/
```

+ Make scripte executable
```console
chmod +x miner_latest.sh
```

+ Make the log file
```console
sudo touch /var/log/miner_latest.log
```

+ By default the script is configured for US915 frequencies. To use the miner in other regios like EU868 the script shall be changed by adding the --env REGION_OVERRIDE=EU868 to the line with docker run.

+ Run the script
```console
./miner_latest.sh
```

+ Configure cron to run the script daily
```console
sudo crontab -e
```

+ Choose "1" to make nano your editor of choice.

+ Add the following at the end of the text editor.

```
# Check for updates on miner image
# Use whatever path you have your repo setup with.
# If you need to test your cron you can use the following site and add "&& curl -s 'https://webhook.site/#!/~'" to the end of your cron and it will make a web request to your specific url when it completes.
0 */4 * * * /home/pi/helium_miner_scripts/miner_latest.sh >> /var/log/miner_latest.log
```

+ Save the file by pressing CTRL-X, and then Y, and then Enter to save changes
