# Homebridge and a Raspberry Pi Home Media Smart Hub
## The Gist
Use the Rasperry Pi as the server and control center for a home media setup. The rPi runs a Homebridge server which spoofs itself as a Homekit device. Set custom configuration in Homebridge to setup controls to switch HDMI inputs and power on/off the projector from iOS devices.
The HDMI switch is RS-232 control signal capable so we're going to opt for that route. The projector requires we use infrared, so we must send IR codes from the rPi and build a IR circuit to "play" the IR signals.


## Ingedients
### Hardware
* Raspberry Pi 2 Model B
* Projector with IR reciever
* 5x1 HDMI Switcher with RS-232 control input

### Software
* [Homebridge](https://github.com/nfarina/homebridge)
* [homebridge-http](https://github.com/rudders/homebridge-http)
* [homebridge-osc](https://github.com/brookstalley/homebridge-osc)
* [forever](https://github.com/foreverjs/forever)


## Installation
### Raspberry Pi setup
Now that I'm using my rPi as a server, I gave it a static IP to make future connection and control simpler.
Read [here](https://www.modmypi.com/blog/how-to-give-your-raspberry-pi-a-static-ip-address-update) to learn how to give your rPi a static IP.
[This post](http://www.howtogeek.com/184310/ask-htg-should-i-be-setting-static-ip-addresses-on-my-router/) describes when to use static IP addresses as well as how to choose the address to set. It turns out I knew nothing about how static addressed worked.

For example, this is the code I addded to my `/etc/dhcpcd.conf`:
```sh
... 
interface wlan0

static ip_address=192.168.0.XXX
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```

### Homebridge
Follow [nfarina](https://github.com/nfarina/)'s [wiki](https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi) on how to install Homebridge on a Raspberry Pi.

Next continue with Homebridge's [normal installation instructions](https://github.com/nfarina/homebridge#installation).

I always want my Homekit server to be running and I want it to run on startup if the rPi is ever shutdown or rebooted so we can used a neat script called [forever](https://github.com/foreverjs/forever). Forever lets us specify a program we want to always be running and restarted if crashed. [This post](http://www.linuxcircle.com/2013/12/30/run-nodejs-server-on-boot-with-forever-on-raspberry-pi/) is a great tutorial on how to setup forever and the run on startup feature.

####Troubleshooting
Forever worked for me straight out of the box, however the startup task took some tinkering. Running `$ homebridge` would work great for me but running `$ sudo homebridge` would end with an error as it could not find the `~/.homebridge/config.json` file as it was looking it in a different directory. I fixed this by making a symlink from the real file to where it was expecting one.
```sh
ln -s ~/.homebridge/config.json /path/to/where/it/wants/config.json
```

At this step I'm assuming Homebridge is working so we can move on to the config file. For my particular setup the controls I am looking for from the Homekit controls on my iPhone are a radio button setup for the HDMI switcher and a standard switch for the projector's power.

As a preface, Homebridge allows for the creation of plugins which define the accessory trying to be made. Both of the 
