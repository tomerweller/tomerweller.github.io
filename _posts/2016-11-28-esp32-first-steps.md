---
layout: post
title:  "ESP32 - first steps"
description: ESP32 - basic setup, programming and benchmarks
draft: true
date:   2016-11-28 23:59:59 -0500
--- 

This post describes my first steps with ESP32. It was originally written as preparation for TA'ing communications week in MIT's [fabclass][fabclass]. It covers basic setup, firmware deployment and a hint of benchmarking. 

*Warning: esp32 is brand spanking new and should be considered a moving target*.

**Contents:** 

 * [Intro](#intro)
 * [Setup](#setup)
    * [Equipment Used](#equipment-used)
    * [Basic Connectivity](#basic-connectivity)
    * [Sanity Check](#sanity-check)
 * [Programming](#programming)
    * [FreeRTOS](#freertos)
    * [ESP - IDF](#esp---idf)
    * [ESP32 ARDUINO CORE](#esp32-arduino-core)
 * [Ring Oscillator benchmark](#ring-oscillator-benchmark)
 * [More Resources](#more-resources)


### Intro

ESP32, successor to the beloved ESP8266, is system on a chip (SoC) that can do (almost) anything:

- Real concurrency with two cores 
- WIFI
- Bluetooth
- BLE
- [Much more][esp32-overview]

![esp32-arch](/assets/esp32/esp32-arch.jpg)

Espressif, the manufacturer, have been kind enough to send some units of their new ESP32 modules for evaluation. (huge thanks to John Lee [@EspressifSystem][espressif-twitter]) This posts explores basic connectivity, different ways to program it and basic benchmarking.

### Setup

####Equipment Used

- [**ESP32 WROOM32**][todo] module. This is Espressif's own ESP32 module. It's safe to assume that we'll see ESP32 used in modules from 3rd party manufacturers in the near future (AI was the lead module manufacturer for the ESP8266).
- **ESP32 Breakout** board. Espressif provided us with simple breakout boards that expose all I/O pins and physical buttons for RESET and BOOT MODE. There are other boards out there and also designs so you can [mill your own][esp32-eagle].
- **FTDI Cable**
- **3.3V Power Supply** (up to 500mA). DO NOT USE THE FTDI POWER - IT CAN'T PROVIDE ENOUGH CURRENT. (TRUST ME - I'VE TRIED)

*Note: soldering the module to the breakout board proved non-trivial, the ground plane is very large so a high wattage soldering iron is recommended for soldering the ground pads.*

####Basic Connectivity

![esp32-pinout](/assets/esp32/esp32-pinout.png)

- VCC (ESP32) -> 3.3V (POWER)
- GND (ESP32) -> GND (POWER)
- GND (ESP32) -> GND (FTDI)
- TX (ESP32) -> RX (FTDI)
- RX (ESP32) -> TX (FTDI)

![esp32-pinout](/assets/esp32/esp32-basic-connectivity.png)

####Sanity Check

1. Open a terminal emulator on the FTDI port with a BAUD rate of 115200
2. Hit the RESET button

{% highlight bash %}
⇒  miniterm.py /dev/tty.usbserial-XXXXXXXX 115200
ets Jun  8 2016 00:22:57
rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
....
IDF version : master(db93bceb)
WIFI LIB version : master(934d079b)
ssc version : master(r283 4d376412)
!!!ready!!!
mode : softAP(26:0a:c4:00:27:f4)
dhcp server start:(ip: 192.168.4.1, mask: 255.255.255.0, gw: 192.168.4.1)
+WIFI:AP_START
{% endhighlight %}

Not only does it work, it even defaults to opening a public access point

{% highlight bash %}
⇒  airport -s | grep ESP
ESP_0027F4 26:0a:c4:00:27:f4 -14  1,+1    Y  -- NONE
When a device enters the network:
n:1 0, o:1 0, ap:1 1, sta:255 255, prof:1
add 1
station: 80:e6:50:27:b6:62 join, AID = 1, g, 20
+SOFTAP:STACONNECTED,80:e6:50:27:b6:62
{% endhighlight %}

I couldn't find any other interesting things in the provided firmware.
In case you were wondering, like me, whether ESP8266's [AT command-set][at-commandset] works in this prompt, the answer is no. It's not very clear what commands do work here.

###Programming
There are currently two methods to program the ESP32: the ESP-IDF and the ESP32 arduino Core. 

####FreeRTOS

Before programming this chip it's crucial to understand that, unlike other embedded systems, the ESP32 comes with a light operating system - FreeRTOS. The following methods to program this chip don't replace the FreeRTOS firmware, but rather deploy applications for it to run. I imagine that in the near future we'll see other operating systems or no-os approaches for reprogramming these chips.  

####ESP - IDF

[Espressif IoT Development Framework][esp32-idf] is a set of open source libraries and tools to facilitate deployment of apps to ESP32s FreeRTOS.

[Code Example : HTTP GET request using the ESP IDF](https://github.com/tomerweller/esp32-rtos-webclient)

####ESP32 ARDUINO CORE

Espressif have also been hard at work to get the maker community happy and makers love Arduino. The [ESP32 arduino core][esp32-arduino-core] integrates ESP-IDF deeply into the arduino tools. This includes providing a WiFi API that is almost 100% compatible with existing wifi shields for arduino.

[Code Example : HTTP GET request using the ESP32 Arduino Core](https://github.com/tomerweller/esp32-arduino-webclient)

*Note: currently, to take advantage of the concurrency capabilities - IDF is the way to go.*

###Ring Oscillator benchmark
Ring oscillator tests measure the minimal amount of time it takes for a signal to get out of a chip and back through peripherals. 

In practice, it means shorting two pins with a jumper, defining one as input and the other as output, and then writing and reading as fast as possible. The result is recorded with an oscilloscope connected to the shortened pins and ground.

Here, I measure both programming methods mentioned earlier:

|FreeRTOS [(code)][rtos-ro]|Arduino Core [(code)][core-ro]|
|:---:|:---:|
| ![rtos-ro](/assets/esp32/esp32-ro-freertos.jpg)1.63MHz|![core-ro](/assets/esp32/esp32-ro-core.jpg)1.05MHz |


**Conclusion:** the Arduino program is about 65% slower than the lower level FreeRTOS program. However, usually in these type of tests, the portability of arduino code comes with a much bigger performance drop. Good job, Espressif!

*Note: These tests only occupy one core. The second core is free to perform other tasks, such as neworking. [Example](https://github.com/tomerweller/esp32-rtos-webclient/tree/with-ring-oscillator).*

###More Resources
- [ESP32 Data Sheet][esp32-datasheet]
- [ESP-WROOM32 Data Sheet][esp32-wroom32-datasheet]
- [ESP32 official forum][esp32-forums]
- [ESP32 Getting started (hackaday)][esp32-getting-started-hackaday]

[todo]:http://www.todo.com
[esp32-overview]:https://espressif.com/en/products/hardware/esp32/overview
[fabclass]:http://fab.cba.mit.edu/classes/863.16/
[espressif-twitter]:https://twitter.com/EspressifSystem
[esp32-datasheet]:https://espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf
[esp32-wroom32-datasheet]:https://espressif.com/sites/default/files/documentation/esp_wroom_32_datasheet_en.pdf
[esp32-forums]:http://www.esp32.com/
[esp32-getting-started-hackaday]:http://hackaday.com/2016/10/04/how-to-get-started-with-the-esp32
[esp32-eagle]:https://github.com/a2retro/ESP32_Miscellany
[at-commandset]:https://www.itead.cc/wiki/ESP8266_Serial_WIFI_Module
[esp32-idf]:https://github.com/espressif/esp-idf
[esp32-arduino-core]:https://github.com/espressif/arduino-esp32
[FreeRTOS]:http://www.freertos.org/
[core-ro]:https://gist.github.com/tomerweller/e50403bb18dcb6932d54e8f11edf0734
[rtos-ro]:https://gist.github.com/tomerweller/7f9f202858cb064c84722c72f6c20aee
