# raspberry-pi-cluster

### Research

Data Sheet suggests DC 5V 2A power

Starting with class 6 cards for benchmarks per suggestion by raspberry pi:
https://www.raspberrypi.org/documentation/installation/sd-cards.md

the following table somewhat confirms this:
http://elinux.org/RPi_SD_cards#SD_card_performance


### Supplies

- 5 Raspberry Pi 2 Model Bs
- 5 High Speed Micro SD Class 6 cards
- 1 Wireless Router with 5 ports or a switch
  - I chose a cheap router with USB input that I can flash with dd-wrt (TP-LINK TL-WR842ND N300 Multi-function Wireless Router)
- 1 Anker PowerPort 5 (40W 5-Port USB Charging Hub)
- 5 Mirco USB Cables
- 5 Cat5e (or better) patch cables -- ethernet port on the rPi2 is limited to 100mbit
- 1 Power strip with at least 2 inputs

To see the exact items I bought check out my [Amazon wish list](http://amzn.com/w/392BF0ANNF6H5)

Total Cost: ~$325

All items were available on Prime at the time of writing this.


### Base Configuration

#### The Router

I modified the router to act as a 5 port switch to start

You need the two files from here:
ftp://ftp.dd-wrt.com/betas/2015/09-28-2015-r27858/tplink_tl-wr842ndv2/

http://www.dd-wrt.com/wiki/index.php/Installation

#### The Pis

Download the latest version of Raspbian:

https://www.raspberrypi.org/documentation/installation/installing-images/README.md


Copy the image to the sd card:

```
df # find out what the card is mounted ass
umount /dev/sdd1 # unmount that card

# copy the image to the card bit by bits
sudo dd bs=4M if=2015-09-24-raspbian-jessie.img of=/dev/sdd


Boot the Pi and upgrade:

sudo apt-get update
sudo apt-get upgrade
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get update
sudo apt-get install vim
```

Run `raspi-config` and configure the following options
- change keyboard language
- resize image to full size of card
- boot to console with login

```
sudo reboot now
```

Backup the card:

```
sudo dd bs=4M if=/dev/sdd of=2015-10-02-pi-cluster-base.img
```

Put the card back in the pi and start copying the image to the 4 other cards:

```
df
umount /dev/sdd1
sudo dd bs=4M if=2015-10-02-pi-cluster-base.img of=/dev/sdd
```

Boot the pi with the first card with the base image, this will become your master:

```
vim /etc/network/interfaces

iface eth0 inet static
address 192.168.2.100
netmask 255.255.255.0
gateway 192.168.2.1
dns-nameservers 192.168.2.1 8.8.8.8 4.2.2.1

sudo sed -i "s/raspberrypi/master/" /etc/hosts
sudo sed -i "s/raspberrypi/master/" /etc/hostname
sudo /etc/init.d/hostname.sh
sudo reboot now
sudo shutdown
```

Backup the card.


Configure the first slave:

```
sudo vim /etc/network/interfaces

iface eth0 inet static
address 192.168.2.101
netmask 255.255.255.0
gateway 192.168.2.1
dns-nameservers 192.168.2.1 8.8.8.8 4.2.2.1

sudo sed -i "s/raspberrypi/s1/" /etc/hosts
sudo sed -i "s/raspberrypi/s1/" /etc/hostname
sudo /etc/init.d/hostname.sh
sudo reboot now
sudo shutdown
```

Backup the card.

### Resources
Video primer on Ansible pi cluster:

https://www.youtube.com/watch?v=ZNB1at8mJWY

Ended up looking into LEDs myself:
https://github.com/geerlingguy/raspberry-pi-dramble/tree/master/images/led-boards

Prefab:
http://www.monkmakes.com/squid/

Github repo:
https://github.com/simonmonk/squid

