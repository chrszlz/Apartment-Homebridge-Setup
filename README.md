# Homebridge and a Raspberry Pi Home Media Smart Hub
## The Gist
Use the Rasperry Pi as the server and control center for a home media setup. The rPi runs a Homebridge server which spoofs itself as a Homekit device. Set custom configuration in Homebridge to setup controls to switch HDMI inputs and power on/off the projector from iOS devices.
The HDMI switch is RS-232 control signal capable so we're going to opt for that route. The projector requires we use infrared, so we must send IR codes from the rPi and build a IR circuit to _play_ the IR signals.


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
