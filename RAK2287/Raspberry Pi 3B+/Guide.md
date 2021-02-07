How to set up rassbery pi 3b+ with rak2267

First get the 64 bit image from https://www.raspberrypi.org/forums/viewtopic.php?t=275370

Then using ether install it on a 64gb sd card

Next enable ssh by making a text file called "ssh.txt"
Then save it to the boot part of the pi.
Edit the file name to "ssh"
	You will need to turn on a windows setting to allow you to save "unstable" file names

Plug in the sd card to the pi, along with a ethernet, and the pi hat, and atennas

open terminal on your computer.

type this `ssh pi@iphere`
	replace iphere with the ip that the pi is running at, you can find this on the routers webpage

the password is `raspberry`

Now you have loaded the os on the pi3B+. 
Update the pi with 'sudo apt upgrade' and 'sudo apt update'

Press Y and enter

Now the fun part!

We need to increase the swap file run thi command 'sudo dphys-swapfile swapoff' and this 'sudo nano /etc/dphys-swapfile'
Edit the line 'CONF_SWAPSIZE=100' to 'CONF_SWAPSIZE=1024"
Press Control-X, and then Y, and finaly enter

Lets reboot now with 'sudo reboot'

Now type in `ssh pi@iphere' to relogin
	replace iphere with the ip that the pi is running at

the password is `raspberry'

We need to enable the SPI and I2C, type in 'sudo raspi-config'

select the 5th one called 'Interfacing Options' by hitting down arrow 4 times, and then enter

select the 5th one called 'IC2' by hitting the down arrow 4 times and then enter

now we got enable it so hit the left arrow and enter

Press enter again

select the 5th one called 'Interfacing Options' by hitting down arrow 4 times, and then enter

select the 5th one called 'SPI' by hitting the down arrow 4 times and then enter

now we got enable it so hit the left arrow and enter

Press enter again

select the 5th one called 'Interfacing Options' by hitting down arrow 4 times, and then enter

select the 5th one called 'serial' by hitting the down arrow 5 times and then enter

now we got disable it so hit the right arrow and enter

now we got enable serial port hardware so hit enter

press enter again
then hit the right arrow twice to select finsih and click enter

press enter to reboot

Now type in `ssh pi@iphere' to relogin
	replace iphere with the ip that the pi is running at

the password is `raspberry'

Update the pi with 'sudo apt upgrade' and 'sudo apt update'

Press Y and enter

We need git so type in 'sudo apt-get install git'

Then run this command 'cd ~'

Update everything just incase
`sudo apt-get update`

Download and run docker
	`sudo apt-get install docker.io`

Then add docker to the usergroup so it can run sudo
	`sudo usermod -aG docker pi`
	
Make a new dir for the docker!
	`mkdir ~/miner_data`

Move into that folder by
	`cd miner_data`

set up docker bu doing these comands

*****BEFORE RUNING THAT MAKE SURE THAT YOU CHANGE THE miner-xxxNN_YYYY.MM.DD to the current one like miner-arm64_2020.09.04.0_GA. *****
They are on quay.io/team-helium/miner. make sure to do arm64 for the pi. 

docker run -d \
--env REGION_OVERRIDE=US915 \
--restart always \
--publish 1680:1680/udp \
--publish 44158:44158/tcp \
--name miner \
--mount type=bind,source=/home/pi/miner_data,target=/var/data \
quay.io/team-helium/miner:miner-arm64_2020.09.04.0_GA


Make sure you have port forwards on your miners '44158/TCP' and '1680/UDP'. Google this step

Now the docker for the miner!
make sure you have a 64gb file


and Then clone the this repo
	`git clone https://github.com/helium/sx1302_hal`

Get into the folder
	`cd sx1302_hal`

Then make it clean it is going to take a min
	`make clean all`

Now you copy a file over from tools dir to the packet_foward directoy
	`cp ./tools/reset_lgw.sh ./packet_forwarder`

Almost done. Now cd into the packet_forwarder
	`cd ./packet_forwarder`

Now we need to make a new file for the config
	`nano global_conf.json`
	
Copy and paste from a window by right clicking onto the terminal from this paste bin
	`https://github.com/maco2035/DiyHeliumHotspots/blob/master/RAK2287/Raspberry%20Pi%203B%2B/global_conf.json`


After it looks the same type `ctrl-x` (like copy in windows). And then `Y` and `enter`.

We have to edit the reset_lgw.sh to make it stop correctly.
	`nano reset_lgw.sh`
	
edit the first 2 lines to this
	SX1302_RESET_PIN=17	
	SX1302_POWER_EN_PIN=18



Now you have done everything!
	Just run this comand your done
	` ./lora_pkt_fwd -c global_conf.json`

Check miner sync
	 docker exec miner miner info height
	 
	 
AUTO UPDATE Script is done on this github
https://github.com/Wheaties466/helium_miner_scripts



Special thanks to Peraly Labs for the free delivery on the rak (everyone gets this)

And the following for the help
Skye#1284, alphamethod#0541, nopeburger#3637, Wheaties466#2588

-maco2035
(maco2035#6152 on discord and maco2035 on git)
Donations are nice, Thank you!
14Cx6WkXKpaD3GFf9VkG8kLsYjQ325y4egFZKs2vJ2hB5Ftrg66


	
