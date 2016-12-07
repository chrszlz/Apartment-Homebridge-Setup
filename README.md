# (rPi + Homebridge) Media Center Home Automation
## The Gist
Use an iOS device and the HomeKit API to control a Raspberry Pi acting as a home media center server-router. The rPi runs [Homebridge](https://github.com/nfarina/homebridge) which is a Node server that spoofs itself as a HomeKit device. Then, define a custom configuration which suits my needs (5x1 HDMI switcher and projector on/off power states). I found an HDMI switch that has RS-232 Serial input and a projector that has an IR remote control which we can emulate with an IR led connected to the rPi’s headphone jack. 


## Ingredients
### Hardware
* Raspberry Pi 2 Model B
* Projector with IR remote control input - [what I bought](https://www.amazon.com/gp/product/B014ULWTVC)
* 5x1 HDMI Switcher with RS-232 control input - [what I bought](https://www.amazon.com/gp/product/B01FXALWYY)

### Software
* [Homebridge](https://github.com/nfarina/homebridge)
	* spoofs the HomeKit device
	* runs on rPi
* [homebridge-switcheroo](https://github.com/chriszelazo/homebridge-switcheroo)
	* Homebridge plugin that make simple switches which trigger http requests
	* traditional on/off switches or radio button multi-switches
* [forever](https://github.com/foreverjs/forever)
	* script that makes sure your servers relaunch if a crash occurs


## Raspberry Pi Setup
### (Optional) Set a Static IP
If you are using your rPi as a server, you may want to give it a static IP. You probably know if you want this, if not, [people have written about it](http://www.howtogeek.com/184310/ask-htg-should-i-be-setting-static-ip-addresses-on-my-router/). 

Example setup that could be added to your `/etc/dhcpcd.conf`:
```sh
... 
interface wlan0

static ip_address=192.168.0.XXX
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```
[(source)](https://www.modmypi.com/blog/how-to-give-your-raspberry-pi-a-static-ip-address-update) 

### Install Homebridge
Follow [nfarina](https://github.com/nfarina/)'s guide on [how to install Homebridge on a Raspberry Pi](https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi).

### Auto-restart Homebridge after a crash
Install [`forever`](https://github.com/foreverjs/forever).

Tell `forever` to run Homebridge, forever... `forever start /usr/bin/homebridge`

Handy commands to have around:
* Stop Homebridge from restarting: `forever stop /usr/bin/homebridge`
* List `forever` servers running: `forever list`

### Run Homebridge (or any Node server) on Startup

`crontab -u pi -e`

Replace `pi` with whatever username you may use (`pi` is generally default).  If you choose another username than yourself, you will have to run with `sudo`. [(source)](http://www.linuxcircle.com/2013/12/30/run-nodejs-server-on-boot-with-forever-on-raspberry-pi/) 

Add the following line:

`@reboot /usr/bin/sudo -u pi -H /usr/local/bin/forever start /usr/bin/homebridge`

Save and exit. Confirm your change:

`crontab -u pi -l `

### Troubleshooting
Forever worked straight out of the box for me; however, the startup task took some work. Running `$ homebridge` would work but running `$ sudo homebridge` would end with an error as it could not find the `~/.homebridge/config.json` file. From what I remember, because the startup launch uses `sudo` it expected to find  `config.json` in a different directory than it was in. I put a bandaid on this by making a symlink from the real file to where it was expecting one.
```sh
ln -s ~/.homebridge/config.json /path/to/where/it/wants/config.json
```


## Homebridge Setup
### The Configuration file
Homebridge’s `config.js` is what defines the /accessories/ that is creates. For me, I need a 5 input radio button / multi-switch and simple binary on/off switch. I also need these switches to make http requests when they are triggered. There were no options to do this when I started this so I wrote a basic switch plugin for this: [`homebridge-switcheroo`](https://github.com/chriszelazo/homebridge-switcheroo). This is my first time writing Javascript so it may be shit, but that’s why it has an MIT license, you can go make it better.

### Install homebridge-switcheroo
`npm install -g homebridge-switcheroo`

### Update your config file
Read [this section of the switcheroo wiki]( https://github.com/chriszelazo/homebridge-switcheroo#configuration-params) . Here’s a sample `config.json`:
```json
{
   "bridge":{
      "name":"Homebridge",
      "username":"CC:22:3D:E3:CE:30",
      "port":51826,
      "pin":"000-00-000"
   },
   "accessories":[
      {
        "accessory": "Switcheroo",
        "switch_type": "Switch",
        "name": "Projector",
        "http_method": "GET",
        "base_url": "http://192.168.0.XXX/projector",
        "on_url": "/state/on",
        "off_url": "/state/off"
      },
      {
        "accessory": "Switcheroo",
        "switch_type": "Multiswitch",
        "name": "Switcher",
        "http_method": "GET",
        "base_url": "http://192.168.0.XXX/switcher/input",
        "multiswitch": [
           "Chromecast",
           "HDMI",
           "Raspberry Pi"
        ]
      }
   ]
}
```


## And I’ll get around to writing the rest of this soon