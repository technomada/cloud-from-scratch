

Check if already installed
```
$ which wg
```
If you don't see something like `/usr/bin/wg` wg is not installed.

If not try install
```
$ sudo apt install wireguard
```

If not try...
```
$ sudo apt-get install raspberrypi-kernel-headers

$ echo "deb http://deb.debian.org/debian/ unstable main" | sudo tee --append /etc/apt/sources.list.d/unstable.list

$ wget -O - https://ftp-master.debian.org/keys/archive-key-$(lsb_release -sr).asc | sudo apt-key add -

$ printf 'Package: *\nPin: release a=unstable\nPin-Priority: 150\n' | sudo tee --append /etc/apt/preferences.d/limit-unstable

$ sudo apt update

$ sudo apt install wireguard 

$ sudo reboot
```
(If have trouble with this step see: https://github.com/technomada/cloud-from-scratch/issues/6)


Verify
```
$ which wg
```
Should see something like
`/usr/bin/wg`

Generate Wireguard Keys
```
$ cd ~
$ mkdir wg-setup
$ cd wg-setup
$ wg genkey | tee privatekey | wg pubkey > publickey
```

Create wg0.conf
```
$ vim wg0.conf
```

Configure, remember to replace the keys and domain name `edge.example.com` with your own.
```
[Interface]
Address = 10.1.1.2/24
PrivateKey = your-LOCAL-NODE-private-key-goes-here=

[Peer]
Endpoint = edge.example.com:51820
PublicKey = your-EDGE-NODE-public-key-goes-here=
AllowedIPs = 10.1.1.1/32, 10.1.1.2/32, 10.1.1.0/24
PersistentKeepalive = 25
```

IMPORTANT KEY PLACEMENTS
Replace `edge.exmaple.com` with your `edge.yourdomain.com` setting from the cloudflare domain setting step.

You'll want to follow this closely it can get tricky...


Copy the **PRIVATE** key of your **LOCAL NODE** to the [Interface] PrivateKey `your-LOCAL-NODE-private-key-goes-here`

Copy the **PUBLIC** key of your **EDGE NODE** to the [Peer] PublicKey `your-EDGE-NODE-public-key-goes-here`

Move wg0.conf to /etc/wireguard
```
$ sudo mv wg0.conf /etc/wireguard
```


(on **EDGE NODE**)
```
$ vim /wireguard/etc/wg0.conf
```

Copy your **PUBLIC** key of your **LOCAL NODE** to the `your-LOCAL-NODE-public-key-goes-here` in your **EDGE NODE** wg0.conf file.
Uncomment '#' 
```
[Peer]
#client 1 -- living room
PublicKey = your-LOCAL-NODE-public-key-goes-here=
AllowedIPs = 10.1.1.2/32
```

(on **EDGE NODE**) restart wireguard
```
$ wg-quick down wg0
$ wg-quick up wg0
```

(on **EDGE NODE**) Check
```
$ wg
```
Should see the something like this (notice the 10.1.1.2/32 entry

```
interface: wg0
  public key: axxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
  private key: (hidden)
  listening port: 51820

peer: Oyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyys=
  allowed ips: 10.1.1.2/32
```

Continue on your **LOCAL NODE**... activate wg
```
$ sudo wg-quick up wg0
```
should see
```
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.1.1.2/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Verify
```
$ sudo wg
```
Should see up and down traffic (if no down traffic verify Endpoint address. if still no, check your keys on both edge and local node wg0.conf files.)

When connected..
on the edge node `$ wg` should look something like
```
interface: wg0
  public key: aaaaaaaaaaaaaaaaaaaaaaaaaaaaa=
  private key: (hidden)
  listening port: 51820

peer: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb=
  endpoint: 299.12.133.4:52001
  allowed ips: 10.1.1.2/32
  latest handshake: 10 seconds ago
  transfer: 180 B received, 92 B sent
```

on the local node `$ sudo wg` should look something like
```
interface: wg0
  public key: ccccccccccccccccccccccccccccc=
  private key: (hidden)
  listening port: 52001

peer: dddddddddddddddddddddddddddddddddd=
  endpoint: 445.176.4.3:51820
  allowed ips: 10.1.1.1/32, 10.1.1.2/32, 10.1.1.0/24
  latest handshake: 3 seconds ago
  transfer: 92 B received, 1.33 KiB sent
  persistent keepalive: every 25 seconds

```
Be sure to note that the received has > 0 bytes.  It may look connected when it's not if received is 0 B.

continuing on **LOCAL NODE**
Enable Wireguard to start automatically at boot
```
$ sudo systemctl enable wg-quick@wg0.service
```

Test
```
$ sudo reboot
$ systemctl status wg-quick@wg0
$ ip a
$ ping 10.1.1.1
```

## Related Links
https://github.com/adrianmihalko/raspberrypiwireguard
