
# Raspberry-PI Server with UPS Module + Power Button + Stats OLED Display

![rpi_server](https://github.com/mmiller1br/mm_projects/assets/32887571/559b215e-a1b6-4b6a-8e24-96157966a5ea)


## RPI Server Pinout

<img src="https://github.com/mmiller1br/mm_projects/assets/32887571/0ecdde58-aee0-4d27-9108-8fafc47352d3" width="400">

**Power IN (from the UPS module):** 
- pin2 (5V PWR) 
- pin6 (GND)
**FAN**: 
- pin1(3.3V)
- pin25(GND)
**Power Button:** 
- pin5 (GPIO3)
- pin9 (GND)
**Display Button:** 
- pin18 (GPIO24)
- pin20 (GND)
**Display:** 
- pin4(5V)
- pin14(GND) 
- pin15(GPIO22) 
- pin16(GPIO23)

## Enable Power Button (GPIO3)

1) edit the file /boot/config.txt
```bash
pi@raspberrypi:~ $
pi@raspberrypi:~ $ sudo nano /boot/config.txt
```

2) add the following line:
```
# enable shutdown button using the default pin5 (GPIO 3)
dtoverlay=gpio-shutdown
```

By adding `dtoverlay=gpio-shutdown` in _config.txt_, we are configuring the Raspberry Pi to use GPIO3 as a startup and shutdown signal.


## Display Statistics

source: https://www.the-diy-life.com/add-an-oled-stats-display-to-raspberry-pi-os-bullseye/

github: https://github.com/adafruit/Adafruit_Python_SSD1306

### installing:
```bash
sudo python -m pip install --upgrade pip setuptools wheel
sudo pip install Adafruit-SSD1306
```

or
```bash
sudo python -m pip install --upgrade pip setuptools wheel
git clone https://github.com/adafruit/Adafruit_Python_SSD1306.git
cd Adafruit_Python_SSD1306
sudo python setup.py install
```

new library: https://learn.adafruit.com/monochrome-oled-breakouts/python-wiring


### check I2C connection

1) edit the file /boot/config.txt
```bash
pi@raspberrypi:~ $
pi@raspberrypi:~ $ sudo nano /boot/config.txt
```
2)  add the following line:
```
# enable 2nd BUS for I2C using GPIO22 (pin 15) and GPIO23 (pin 16)
dtoverlay=i2c-gpio,bus=3,i2c_gpio_sda=22,i2c_gpio_scl=23
```

This will create a new BUS for the I2C, so we can connect the display using the new BUS, using the GPIO pins specified by this command. 

⚠️ This is necessary once the default GPIO3 will be used for the Power Button. ⚠️

Connect your display using the pins GPIO22 and GPIO23 as your pins SDA and SCL, and test using the following command:
```bash
pi@raspberrypi:~ $ i2cdetect -y 3
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- 3c -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
pi@raspberrypi:~ $
```

The example above shows the display detect on BUS3.


## Enable stats using crontab:

@reboot python3 /home/pi/Adafruit_Python_SSD1306/examples/statsv3.py &


## Python script

Check the statsv3.py for more details about the script used on this project.
