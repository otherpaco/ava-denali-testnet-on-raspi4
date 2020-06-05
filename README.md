# ava-denali-testnet-on-raspi4
short how to and notes how to run an ava node and validator on a RaspberryPi4

## Buylist

- Rapsberry Pi 4B 8GB RAM
- Micro SD card 128GB SanDisk Ultra
- official Power Supply USB-C DC 5.1V 3A
- Network cable or use WLAN
- cooling case with or without fan, dont know yet
- micro HDMI to something cable if you need a screen or use ssh

smaler sd cards should work  
sanDisk Ultra because I never had failure with them in any RasPi

## Ubuntu 20.4

https://ubuntu.com/download/raspberry-pi

Ubuntu 20.04 LTS 64-bit for RasPi4

- check the checksum after download
- unxz file
- dd the image to your sd card
- use duckduckgo.com or google for more info for above points
- https://linuxhint.com/raspberry_pi_headless_mode_ubuntu/ to setup ssh directly so you dont need a display
- short add a file "ssh" in the boot directory
- put sd card in RasPi
- plug network cable in RasPi and Router
- plug in power supply
- wait and use another computer in the same network to connect to RasPi
- ssh ubuntu@ubuntu
- user: ubuntu
- pw: ubuntu (you will be asked to change that)
- for securtiy (hardening) use the internet
- wait for ubuntu to self update a bit then
- sudo apt update
- sudo apt full-upgrade

add some extra GB Memory aka swap

from here https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-18-04

- sudo fallocate -l 16G /swapfile
- sudo chmod 600 /swapfile
- sudo mkswap /swapfile
- sudo swapon /swapfile
- sudo cp /etc/fstab /etc/fstab.bak (just in case)
- you can work on swapinees etc
