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

or press `F9` in byobu and configure it to show temperature in the info bar 

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

### NOW WE WAIT AND GET FRUSTRATED - DON'T ... GET FRUSTRATED BUT BE PATIENT

"BOOTSTRAPPING" it is all over the Discord Channel and it sucks, a bit as it takes some time ...  
What you will see after you entered `./ava`  
Some ASCII-Art and then:

```sh
WARN [06-05|19:20:35] /main/main.go#44: NAT traversal has failed. If this node becomes a staker, it may lose its reward due to being unreachable.
WARN [06-05|19:20:35] /main/main.go#65: assertions are enabled. This may slow down execution
...
INFO [06-05|19:20:35] /api/server.go#105: adding route /ext/admin
INFO [06-05|19:20:35] /node/node.go#475: initializing Health API
INFO [06-05|19:20:35] /api/server.go#105: adding route /ext/health
INFO [06-05|19:20:35] /chains/manager.go#187: creating chain:
INFO [06-05|19:20:35] <chain 11111111111111111111111111111111LpoYY> /vms/platformvm/vm.go#695: next scheduled event is at 2020-10-31 00:00:00 +0000 UTC (3532h39m24.274439418s in the future)
```
Ignore both warnings for the time beeing https://docs.ava.network/v1.0/en/faq/faq/#node-prints-nat-traversal-failed

Now idle idle sometimes 20 minutes or 1 hour

Oh did you start ava in byobu? No? Then the moment you close your ssh connection all is lost und you have to start again with `./ava` and bootstrapping. We will make a process that runs on its own in the end. ;-)

```sh
INFO [06-05|20:28:56] <chain 11111111111111111111111111111111LpoYY> /snow/engine/common/bootstrapper.go#132: Bootstrapping finished with no accepted frontier. This is likely a result of failing to be able to connect to the specified bootstraps, or no transactions
```

WHAAAT something failed ? No all good

```sh
INFO [06-05|20:28:56] <chain abcdefghfwrfgiogjtgojionevinviogoeghgeggebbbebehg> /snow/engine/snowman/transitive.go#47: Initializing Snowman consensus
...
INFO [06-05|20:28:56] <chain Hkfgk76fghvertfsQgmfhstTvneduifsdf5hGtuAsCYugdFgg> /snow/engine/avalanche/transitive.go#43: Initializing Avalanche consensus
...
INFO [06-05|20:28:56] <chain fgggfgdfgse5u46dfhtjxdHKTODIRsgegdgdfgt5fGERGCHrhg> /snow/engine/snowman/transitive.go#47: Initializing Snowman consensus
...
INFO [06-05|20:28:56] <chain amrghfgxct/ikgdrgujpgks5rsdgx325vdgdgKwfhgd5KHAhtM> /snow/engine/snowman/transitive.go#47: Initializing Snowman consensus
```

Some consensus mechanics were initialized - good

the something about bootstrapping

```sh
INFO [06-05|20:29:00] <chain 1GHaaPbg1YBR4qiATuFNfL2j9StJwmddwXiwm1s9ts5KXvMai> /snow/engine/avalanche/transitive.go#70: Bootstrapping finished with 0 vertices in the accepted frontier
...
INFO [06-05|20:29:00] <chain zJytnh96Pc8rM337bBrtMvJDbEdDNjcXG3WkTNCiLp18ergm9> /snow/engine/snowman/transitive.go#99: Bootstrapping finished with BhdXsMYpJsjGdgk1wqUWcukurCovwv9hpKMuLv7bJ9jVeSZJj as the last accepted block
```

sometimes 0, sometimes more vertices - all goog

then a lot of this

```sh
2020-06-05T20:37:05.738Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:05.738] FS scan times                            list=194.72µs   set=2.5µs    diff=2.481µs
2020-06-05T20:37:08.739Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:08.739] FS scan times                            list=247.83µs   set=4.092µs  diff=14.278µs
...
2020-06-05T20:37:32.744Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:32.744] FS scan times                            list=220.201µs  set=2.907µs  diff=2.574µs
2020-06-05T20:37:35.744Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:35.744] FS scan times                            list=186.441µs  set=2.575µs  diff=2.259µs
2020-06-05T20:37:36.831Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:36.830] Persisted trie from memory database      nodes=4 size=591.00B time=1.976214ms gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
2020-06-05T20:37:36.835Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:36.834] Inserted new block                       number=1 hash=2e68fe…9535bf uncles=0 txs=1 gas=21000 elapsed=12.285ms root=95f8a8…2a97db
...
2020-06-05T20:37:36.835Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:36.835] Reinjecting stale transactions           count=0
2020-06-05T20:37:36.849Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:36.848] Persisted trie from memory database      nodes=4 size=623.00B time=1.432925ms gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
...
2020-06-05T20:37:59.750Z [DEBUG] plugin.sh: DEBUG[06-05|20:37:59.750] FS scan times                            list=263.478µs  set=4.722µs  diff=4.5µs
2020-06-05T20:38:02.750Z [DEBUG] plugin.sh: DEBUG[06-05|20:38:02.750] FS scan times                            list=153.109µs  set=2.13µs   diff=2.278µs
...
```

At the moment the CPU has ~61°C, caselees at the moment. How do I know? Read more...

### MONITOR

We can monitor stuff now:

byobu new window `F2`, you can have a lot of them

- Temperature  
  `watch cat /sys/class/thermal/thermal_zone0/temp`
  or byobu status bar (config it with `F9`)
- DB folder  
  `watch ls -al ~/.gecko/db/denali/v0.5.0/`
- size of DB folder  
  `watch du -hs ~/.gecko/db/denali/v0.5.0/`
- ava process  
  `ps -aux | grep ava` or
  `ps aux | sed -e '1b' -e '/ava/!d'`
- Memory  
  `watch free -h`

In the `~/.gecko` folder lives a lot of "personal" gecko stuff.

## IN THE MEANTIME

### create an address X-Chain

```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :2,
    "method" :"avm.createAddress",
    "params" :{
        "username": "YOUR USERNAME",
        "password": "YOUR PASSWORD"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

response is something like this

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "address": "YOUR X-CHAIN ADDRESS"
    },
    "id": 1
}
```

**write down your address**

### create an address P-Chain

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAccount",
    "params": {
        "username": "YOUR USERNAME",
        "password": "YOUR PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

if you get a 404 just wait, early during bootstrapping it fails see https://docs.ava.network/v1.0/en/faq/faq/#why-am-i-getting-a-404-when-i-make-an-api-call

response

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "address": "YOUR P-CHAIN ADDRESS"
    },
    "id": 1
}
```

**write it down**

but you can request it with

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "avm.listAddresses",
    "params": {
        "username":"myUsername",
        "password":"myPassword"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

## SEND nAVA TO THE PLATFORM P-CHAIN OR STAKE

still using this link https://medium.com/avalabs/how-to-join-avas-denali-test-network-9bbfb353207b

```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"avm.exportAVA",
    "params" :{
        "username": "YOUR USERNAME",
        "password": "YOUR PASSWORD",
        "to":"YOUR PLATFORM ADDRESS HERE",
        "amount": 10000
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

Write that txID down, too. The first time noting happened and when I checked the `avm.getTxStatus` it was "Unknown".
I checke my balance that was still 20.000.

https://docs.ava.network/v1.0/en/api/avm/#avmgettxstatus

https://docs.ava.network/v1.0/en/api/avm/#avmgetbalance

So I sent it a secodn time. It staid the same txID, this time after one second the status was "Processing".

After a few hours it was "Accepted".

### Accept the tranfer on P-Chain

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.importAVA",
    "params": {
        "username": "YOUR USERNAME",
        "password": "YOUR PASSWORD",
        "to":"YOUR PLATFORM ADDRESS HERE",
        "payerNonce":1
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/P
```

**note** the returned tx value, a long string.

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
        "tx":"THE ISSUE TRANSFER TX HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/P
```

Take the resulting transaction and issue it to the P-Chain from the API call "issueTx"

Took nearly a day but the funds arrived on the P-Chain
Check your AVA balance in the P-Chain account.

```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"platform.listAccounts",
    "params" :{
        "username": "YOUR USERNAME",
        "password": "YOUR PASSWORD"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

## BECOME A VALIDATOR

This will give you your nodeID

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "admin.getNodeID",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

Now add your nodeID to the network

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addDefaultSubnetValidator",
    "params": {
        "id":"YOUR NODEID HERE",
        "payerNonce":2,
        "destination":"YOUR PLATFORM ADDRESS HERE",
        "startTime":'$(date --date="15 minutes" +%s)',
        "endTime":1592265599,
        "stakeAmount":10000,
        "delegationFeeRate":0
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

Platform address is your P-address to where the money returns after staking.
Time is in Unix timestamp, seconds sice Jan 1st 1970.
Here is a converter page https://www.unixtimestamp.com/
You can use a Timestamp for the startTime, too.

You'll get an unsigned transaction as response like this:

```sh
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "unsignedTx": "1115K3jV5Yxr145wi6kEYpN1nPz3GEBkzG8mpF2s2959VsR54YGenLJrgdg3UEE7vFPNDE5n3Cq9Vs71HEjUUoVSyrt9Z3X7M5sKLCX5WScTcQocxjnXfFowZxFe4uH8iJU7jnCZgeKK5bWsfnWy2b9PbCQMN2uNLvwyKRp4ZxcgRptkuXRMCKHfhbHVKBYmr5e2VbBBht19be57uFUP5yVdMxKnxecs"
    }
}
```

We have to sign it

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
        "username": "YOUR USERNAME HERE",
        "password": "YOUR PASSWORD HERE",
        "tx":"THE VALIDATION UNSIGNED TX HERE",
        "signer":"YOUR PLATFORM ADDRESS HERE"
    },
    "id": 2
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

response is like that:

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "111Bit5JNASbJyTLrd2kWkYRoc96swEWoWdmEhuGAFK3rCAyTnTzomuFwgx1SCUdUE71KbtXPnqj93KGr3CeftpPN37kVyqBaAQ5xaDjr7wVBTUYi9iV7kYJnHF61yovViJF74mJJy7WWQKeRMDRTiPuii5gsd11gtNahCCsKbm9seJtk2h1wAPZn9M1eL84CGVPnLUiLP"
    },
    "id": 1
}
```

last step is to issue it to the P-Chain

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
        "tx":"YOUR VALIDATION SIGNED TX HERE"
    },
    "id": 3
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

When you call this:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 4
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

you should find your nodeID and P-address in the list of pending validators.

or a little later this call:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getCurrentValidators",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

gives you the list of current validators. That is the list you want to be in.

You can check https://explorer.ava.network/ there is a validator searchbar.

If the last steps are not working stop ./ava, restart it and repeat from `platform.addDefaultSubnetValidator` or maybe from the `platform.issueTx` that you did before that call.

Good luck have fun.

## Clean Up

Let's create a dedicated user just for gecko

```sh
sudo adduser --gecos ',,,' --disabled-password --home /opt/gecko gecko
```

Whenever we need this user we get it with `sudo su gecko`. The user itself has no password and can not login.

When you did not start yet, start here and install gecko in this folder `/opt/gecko` otherwise you have to copy a lot of stuff and change the owner etc etc set the $GOPATH and so on. You may want to stick to your installation.  
In the end this will be a complete guide and this section will be in the beginning.




## TODOS:
systemd
