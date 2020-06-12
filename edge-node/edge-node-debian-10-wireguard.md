## Debian 10 Wireguard Edge Node Setup

Location: Follow these instructions on your **EDGE NODE**

("Users with Debian releases older than Bullseye should enable backports." This includes Buster so, we'll do backport.)


Edit sources.list
```
$ vim /etc/apt/sources.list.d/sources.list
```

add the following to sources.list
```
deb https://deb.debian.org/debian buster-backports main
```

install wireguard
```
$ apt update
$ apt -t buster-backports install "wireguard"

$ reboot
```
Reconnect to server after reboot.

Verify Install
```
$ which wg
```
(should see something like `/usr/bin/wg`)

Generate Wireguard Keys
```
$ cd ~
$ mkdir wg-setup
$ cd wg-setup

$ wg genkey | tee privatekey | wg pubkey > publickey
```
(contents of privatekey or public key file look like `ayQFJiofd+fji2oDi8N/Jfi3=`)

Create wg0.conf file
```
$ vim wg0.conf
```
We're using `10.1.1.1/24` from here forward you may use this or your own preference for the wireguard network.  ListenPort may also be your choice.  Here we'll use `51820`.
```
[Interface]
Address = 10.1.1.1/24
ListenPort = 51820
PrivateKey = your-private-key-text-from-privatekey-genearation=

#[Peer]
##client 1 -- living room
#PublicKey = clients-public-key-from-a-future-step=
#AllowedIPs = 10.1.1.2/32

#[Peer]
##remote pi 1 -- kitchen
#PublicKey = clients-public-key-from-a-future-step= 
#AllowedIPs = 10.1.1.3/32
```

Copy the `~/wg-setup/wg0.conf` file to `/etc/wireguard/wg0.conf`
```
$ cp wg0.conf /etc/wireguard/wg0.conf
```

We'll come back and fill out clients-public-key in the `/etc/wireguard/wg0.conf` file in an upcomming step.

Start Wireguard
```
$ wg-quick up wg0
```

Should see something like
```
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.1.1.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```
If you see something like `Protocol not supported` make sure you've rebooted.

Verify Looks good
```
$ wg
```
should see someting like
```
interface: wg0
  public key: sf8fjDJj9fiJaawQdfiDFi8sfjkl+fxXIxf/=
  private key: (hidden)
  listening port: 51820
```

Enable Wireguard to start automatically at boot
```
$ systemctl enable wg-quick@wg0.service
```

Reboot to verify its working.
```
$ reboot
```

Verify
```
$ systemctl status wg-quick@wg0
```

Test
```
$ ip a ... (should show wg0 interface)
$ wg   ... (should show client config)
```

