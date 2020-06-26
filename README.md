# Run gecko on a RaspberryPi and make it an AVA node and validator

AVA announced the Denali [Testnet Challenge](https://www.avalabs.org/denali) Required Hardware requirements were:

- 2 GHz or faster CPU, 3 GB RAM, 250 MB hard disk.
- OS: Ubuntu >= 18.04 (works with Windows WSL) or macOS >= Catalina.

So I thought let's use a RasPi for that. It is a little short GHz wise with only 1.5 but we will see.

So here is my buylist.

## Buylist

- Rapsberry Pi 4B 8GB RAM (2 and 4GB RAM should work, too)
- Micro SD card 128GB SanDisk Ultra (64GB should be enough for now)
- official Power Supply USB-C DC 5.1V 3A
- network cable
- Armor Case passive cooling
- someday I might need an external SSD but not yet

## Install Ubuntu 20.4

First we need to install a 64bit system. Raspian is a 32bit system so we can not use it.
Best choice is Ubuntu.  
[Download Ubuntu 20.04 LTS 64-bit for RasPi4](https://ubuntu.com/download/raspberry-pi)  
If you don't know how to get the image on the SD-card follow the [tutorial](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview).

## Set up ssh

Because we don't want to plug in a keyboard and screen all the time we will use ssh to connect to the RasPi. Therefore we have do create a file called `ssh` in the boot partion. Following this [tutorial](https://linuxhint.com/raspberry_pi_headless_mode_ubuntu/).

Now we can connect us to the Raspi from any computer in the same network with `ssh ubuntu@ubuntu`. The initial password is `ubuntu`.  
Securing your RasPi, e.g. using keys for ssh or unattended-upgrades, is recommended but not part of this tutorial.
You should update your RasPi with `sudo apt update` followed by `sudo apt full-upgrade`.

## Add some extra GB Memory aka swap

Especially when you use a RasPi with less than 3GB RAM you should give the system some swap, that is a kind of extra RAM on your SD-card. With the 8GB RasPi you can skip this part.

I used this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-18-04) to add 8GB. Normaly you should make it x1 to x2 the size of your RAM.

```sh
sudo fallocate -l 8G /swapfile #8G is the size here 8GB
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo cp /etc/fstab /etc/fstab.bak #just in case a backup
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## byobu

```sh
sudo apt install byobu #should already be installed
```

byobu is the perfect shell if you want to use stuff like `nohub`, `screen` or `tmux` but the easy-way.
It is the perfect shell when you connected via ssh, as it runs in the background even when your connection breaks down.

From here on I **ALWAYS** work in **byobu** on my RasPi. When I get disconnected the process keeps on running and when I come back and type `byobu` I am where I left, only exception is when the RasPi was rebooted in the meantime.

After you connect to your RasPi just type `byobu`. Here are some [tips](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-byobu-for-terminal-management-on-ubuntu-16-04) how to use it.

## Create an user for gecko

```sh
sudo adduser --gecos ',,,' --disabled-password --home /opt/gecko gecko
```

Whenever we need this user we get it with `sudo su gecko`. The user itself has no password and can not login. So we login as `ubuntu` and switch to `gecko` when needed.

The $HOME folder for this user is `/opt/gecko`.

**For the following steps I will use the user `gecko` with `sudo su gecko`.**

## Install AVA/gecko

I used this [tutorial](https://medium.com/@nitaxmx/ava-testnet-validator-in-raspberry-pi-59bd16b59d64).

First we install the language go with `sudo apt install golang`.

Check the version with `go version` we need go1.13.x or higher.

We continue with some stuff taken from [ava-labs github](https://github.com/ava-labs/gecko#installation).

### $GOPATH

The GOPATH should already be set ([golang github](https://github.com/golang/go/wiki/SettingGOPATH)) but somehow **it is not**.  
We still use the `gecko` user.  
Try `echo $GOPATH` if you see a blank line use the following commands to set the GOPATH.

- `cd ~`
- `mkdir go`
- `nano .bashrc`
- add `export GOPATH=$HOME/go` and an empty line to the end of the file
- save the file and close nano
- `source .bashrc`
- `echo $GOPATH` should return `/opt/gecko/go`

### gecko

Still user `gecko` still using `byobu`.

Let's build gecko from source.

```sh
sudo apt-get install libssl-dev libuv1-dev cmake make curl g++
go get -v -d github.com/ava-labs/gecko/...
cd $GOPATH/src/github.com/ava-labs/gecko
./scripts/build.sh
```

## Join the network

I follow mostly this [article](https://medium.com/avalabs/how-to-join-avas-denali-test-network-9bbfb353207b).

```sh
cd $GOPATH/src/github.com/ava-labs/gecko/build
./ava
```

While gecko is running, keep it running and open another window `F2` in byobu (`F3` and `F4` to switch windows, `exit` to close, `F6` to detach byobu you can come back anytime just type `byobu`)

In the new window we make an API call to create an user.

### Create an user

```sh
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "YOUR USERNAME",
         "password": "YOUR PASSWORD"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

You will get a response like this:

```sh
response {"jsonrpc":"2.0","result":{"success":true},"id":1}
```

[Documentation](https://docs.avax.network/v1.0/en/api/keystore/#keystorecreateuser)

NOW WE WAIT AND GET FRUSTRATED - DON'T ... GET FRUSTRATED BUT BE PATIENT

Your node is bootstrapping and this may take a little while up to a few hours with bad luck. But this will only take so long on your very first start or when the node has been offline a long time.  

What you will see after you entered `./ava`  
Some ASCII-Art and then:

```sh
WARN [06-05|19:20:35] /main/main.go#44: NAT traversal has failed. If this node becomes a staker, it may lose its reward due to being unreachable.
...
INFO [06-05|19:20:35] /api/server.go#105: adding route /ext/admin
INFO [06-05|19:20:35] /node/node.go#475: initializing Health API
INFO [06-05|19:20:35] /api/server.go#105: adding route /ext/health
INFO [06-05|19:20:35] /chains/manager.go#187: creating chain:
INFO [06-05|19:20:35] <chain 11111111111111111111111111111111LpoYY> /vms/platformvm/vm.go#695: next scheduled event is at 2020-10-31 00:00:00 +0000 UTC (3532h39m24.274439418s in the future)
```

Ignore the warning for the time beeing [FAQ](https://docs.avax.network/v1.0/en/faq/faq/#node-prints-nat-traversal-failed)

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

sometimes 0, sometimes more vertices - all good

How do you know your node finished bootsptrapping? [FAQ](https://docs.avax.network/v1.0/en/faq/faq/#is-my-node-done-bootstrapping)

### Monitor stuff

We can monitor stuff.

Open new windows in byobu `F2`, you can have a lot of them:

- Temperature  
  `watch cat /sys/class/thermal/thermal_zone0/temp`
  or in the byobu status bar (config it with `F9`)
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

The `watch` command repeats the following command every 2 seconds, you can exit with `Strg + c`.

### Create an address on the X-Chain

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

Response is something like this:

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "address": "YOUR X-CHAIN ADDRESS"
    },
    "id": 1
}
```

[Documentation](https://docs.avax.network/v1.0/en/api/avm/#avmcreateaddress)

Write down your address.

but you can request it with:

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

[Documentation](https://docs.avax.network/v1.0/en/api/avm/#avmlistaddresses)

Now you should get yourself some AVA from [faucet](https://faucet.avax.network/). Use your X-address.

### Create an address on the P-Chain

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

If you get a 404 your node is probably still bootstrapping [FAQ](https://docs.avax.network/v1.0/en/faq/faq/#why-am-i-getting-a-404-when-i-make-an-api-call)

Response:

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "address": "YOUR P-CHAIN ADDRESS"
    },
    "id": 1
}
```

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformcreateaccount)

Write it down.

but you can request it with

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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformlistaccounts)

## Send AVA to the Platform P-Chain

Still using this article ["how-to-join-avas-denali-test-network"](https://medium.com/avalabs/how-to-join-avas-denali-test-network-9bbfb353207b)

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

[Documentation](https://docs.avax.network/v1.0/en/api/avm/#avmexportava)

Write that txID down, too. The first time noting happened and when I [checked the TxStatus](https://docs.avax.network/v1.0/en/api/avm/#avmgettxstatus) it was "Unknown".  
I [checked my balance](https://docs.avax.network/v1.0/en/api/avm/#avmgetbalance) and it was still 20.000.

I sent it a second time. It staid the same txID, this time after one second the status was "Processing".
After a few hours it was "Accepted". Should be much faster now.

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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformimportava)

Take a **note** of the returned tx value, a long string.

Whenever you send a request that has the field `payerNonce` it must be one up of your recent nonce for the account (P-Address).  
[How to check your nonce (platformgetaccount)](https://docs.avax.network/v1.0/en/api/platform/#platformgetaccount) or  
[How to check your nonce (platformlistaccounts)](https://docs.avax.network/v1.0/en/api/platform/#platformlistaccounts)

Take the resulting transaction from `importAVA` and issue it to the P-Chain with API call `issueTx`.

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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformissuetx)

Took nearly a day but the funds arrived on the P-Chain. This should be faster, too now.  
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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformlistaccounts)

## Become a validator

You need your nodeID

```sh
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

[Documentation](https://docs.avax.network/v1.0/en/api/info/#infogetnodeid)

Now add your nodeID to the network:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addDefaultSubnetValidator",
    "params": {
        "id":"YOUR NODEID HERE",
        "payerNonce":2,
        "destination":"YOUR PLATFORM ADDRESS HERE",
        "startTime":'$(date --date="15 minutes" +%s)',
        "endTime":'$(date --date="15 days" +%s)',
        "stakeAmount":10000,
        "delegationFeeRate":0
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

Platform address is your P-address to where the money returns after staking.  
Time is in Unix timestamp or in other words seconds sice Jan 1st 1970.  
Here is a [converter](https://www.unixtimestamp.com/).  
You can use a Timestamp for the startTime and endtime directly.  
Now it will calculate start time 15 minutes in the future and end time 15 days in the future. Maximum staking duration is one year minimum one day, minimum will change to 14 days.  
Because we start 15 minutes in the future we will validate for 14 days 23 hours and 45 minutes.

You'll get an unsigned transaction as response like this:

```sh
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "unsignedTx": "1115K3sV5Yxr175wi6kEYpN1nPz3GEBkzG8mpF2s2959VsR54YGenLJrgdg3UEE7vFPNDE5n3Cq9Vs71HEjUUoVSyrt9Z3X9M9sKLCH5WScTcTocxj1XfFowZxFe4uH8iJU7jnCAgeKK5bAsfnWy2b9PbCQMN2uNLvwyKRp4ZxcgRptkuXRMCKHfhbHVKBYmr5e2VbABht19be57uFUP5yVdMxKnxecs"
    }
}
```

We have to sign it:

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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformsign)

Response looks like this:

```sh
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "111Bit7JNASbJyTLrd2kWkYRoc96swEWoWdmEhuGAFK3rCAyTnTzomuFwgx1SCUdUE71KbtXPnqj93KGr3CeftpPN37kVyqBaAQ5xaDjr7wVABCCi9iV7kYJnHF61yovViJF69mJJy7WWQKeRMDRTiDuii5gsd11gtNahCCsKbm9seJtk2h1wAPZn9M1eL84CGVPnLUiLP"
    },
    "id": 1
}
```

Last step is to issue it to the P-Chain:

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

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformissuetx)

When you call this:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 4
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformgetpendingvalidators)

you should find your nodeID and P-address in the list of pending validators.

Or a little later this call:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getCurrentValidators",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

[Documentation](https://docs.avax.network/v1.0/en/api/platform/#platformgetcurrentvalidators)

Gives you the list of current validators. That is the list you want to be in.

You can check the [network explorer](https://explorer.avax.network/) there is a validator searchbar.

If the last steps are not working stop `./ava`, restart it and repeat from `platform.addDefaultSubnetValidator` or maybe from the `platform.issueTx` that you did before that call.

Good luck have fun.

## systemd

How to start gecko on every startup and keep it alive in the background.

Inspired by this [article](https://medium.com/@dogusural/how-to-keep-denali-node-alive-on-linux-c16d57e561ca).

Check the article above for info how to monitor your node. I changed the name of the service to `gecko`. So you have to replace "denali" with "gecko" when you use commands from the article above.

We change back to our regular user `ubuntu`. When you are user `gecko` just type `exit` and you should be `ubuntu` again.

```sh
sudo touch /etc/systemd/system/gecko.service
```

as sudo enter the following lines to the file with the editor of your choice, e.g. `sudo nano /etc/systemd/system/gecko.service`

```sh
[Unit]
Description=gecko AVA node
After=network-online.target
Requires=network-online.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=gecko
SyslogIdentifier=gecko.service
ExecStart=/opt/gecko/go/src/github.com/ava-labs/gecko/build/ava
[Install]
WantedBy=multi-user.target
```

Start service and enable it so it starts after reboot.

```sh
sudo systemctl daemon-reload
sudo systemctl start gecko.service
sudo systemctl enable gecko.service
```

## Backup your stuff

[backup your node](https://medium.com/@otherpaco/backup-your-precious-ava-node-85650b9941a7)

## API calls from another machine

We made all our API calls from the machine gecko is running on. When you want to do it from another machine in your local network you have to make sure your RasPi is getting the same local IP everytime and start gecko with this IP `.ava --http-host=123.my.ip.45`.  
[Documentation](https://docs.avax.network/v1.0/en/references/command-line-interface/)

If you use this method or other arguments and options make sure to add them to `ExecStart` in the systemd `gecko.service`.

```sh
...
SyslogIdentifier=gecko.service
ExecStart=/opt/gecko/go/src/github.com/ava-labs/gecko/build/ava --http-host=123.my.ip.45
[Install]
...
```

You should `sudo systemctl daemon-reload` and restart the `gecko.service`.

It should be possible to do it with your public IP and some port forwarding but I did not try that. I would prefer to connect to my local network via VPN and make the calls "locally".

You should be all set!

You can find me and some other people running a node on a RasPi in the [AVA Discord channels](https://chat.avalabs.org/).
