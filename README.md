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

### Ubuntu 20.04 LTS 64-bit for RasPi4

- check the checksum after download
- unxz file
- dd the image to your sd card
- use duckduckgo.com or google for more info for above points
- https://linuxhint.com/raspberry_pi_headless_mode_ubuntu/ to setup ssh directly so you dont need a display
- short add a file `ssh` in the boot directory
- put sd card in RasPi
- plug network cable in RasPi and Router
- plug in power supply
- wait and use another computer in the same network to connect to RasPi
- `ssh ubuntu@ubuntu`
- user: ubuntu
- pw: ubuntu (you will be asked to change that)
- for securtiy (hardening) use the internet, eg ssh key instead of pw and so on
- wait for ubuntu to self update a bit then
- `sudo apt update`
- `sudo apt full-upgrade`

### add some extra GB Memory aka swap

from here https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-18-04

```sh
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo cp /etc/fstab /etc/fstab.bak #just in case a backup
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
you can work on swapinees etc

### add some tools some you don't need but love

```sh
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
sudo apt install byobu (should already be there)
```

zsh top shell  
ohmyzsh top config for zsh  
byobu good shell if you use ssh as it runs even when you disconnect, you can zsh inside if you want

from her on I **ALWAYS** start **byobu** in my shell so when I get disconnected the process keeps om running and when I come back and type byobu I am where I left, except for reboots of the Pi.

## GECKO

I follow mostly this one  
https://medium.com/@nitaxmx/ava-testnet-validator-in-raspberry-pi-59bd16b59d64

- `sudo apt install golang`
- `go version` should show go1.13.x or higher

from here on https://github.com/ava-labs/gecko#installation

### GOPATH

- `mkdir go`
- GOPATH should already be set https://github.com/golang/go/wiki/SettingGOPATH but somehow **is not**
- `echo $GOPATH` shows nothing, blank line
- `nano .bashrc` add "export GOPATH=$HOME/go" and empy line at end of file
- `source .bashrc`  you need to be in the bash for this
- `echo $GOPATH` gives you `/home/ubuntu/go`
- same for `.zshrc` you need to be in zsh for the `source .zshrc`command

### GECKO

```sh
go get -v -d github.com/ava-labs/gecko/...
cd $GOPATH/src/github.com/ava-labs/gecko
./scripts/build.sh
```

DO IT IN BYOBU just in case you loose ssh connection!!

## MONITOR TEMPERATURE

`cat /sys/class/thermal/thermal_zone0/temp`

gives you the CPU temp, divide it by 1000 and you have degrees Celsius

`watch cat /sys/class/thermal/thermal_zone0/temp`

refreshes the value every 2 seconds `Strg + c` exits

## JOINING THE CHALLENGES

https://medium.com/avalabs/how-to-join-avas-denali-test-network-9bbfb353207b

if you can, open port 9651 to the RasPi in your router.

```sh
cd $GOPATH/src/github.com/ava-labs/gecko/build
./ava
```

while this is running, keep it running open another window `F2` in byobu (`F3` and `F4` to switch windows, `exit` to close, `F6` to detach byobu you can come back anytime `byobu`)

### CREATE USER

```sh
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "user",
         "password": "password"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

response {"jsonrpc":"2.0","result":{"success":true},"id":1}



## TODOS:

set ubuntu timezone 

