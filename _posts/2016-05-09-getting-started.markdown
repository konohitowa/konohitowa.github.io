---
layout: default
title: Getting Started
---
This section covers the hardware and software tools you'll need to follow along. I'll cover what I used and try to point out alternatives if I'm aware of them. Constructive comments are always welcome.

If you haven't read the [about](/about.html), do that first. It contains links to the originating projects used on this blog and other details.

### Note
---
When I first started this project, I was using my MacBook as a host (OS X.11) and built the aarch64-none-elf toolchain to cross compile my code. I was able to do this because I was using C. When I switched to Rust, I had to switch to the aarch64-linux-gnu toolchain because the Rust compiler doesn't currently have support for aarch64-none-elf, and I don't know enough about the compiler to add that support. After a lot of hours trying to get aarch64-linux-gnu to build -- I even resorted to crosstool-ng which seemed like having to hire the *Geek Squad* to come fix my computer -- I finally gave in and switched to my Ubuntu VM. My instructions will be targeted toward Ubuntu 14.04 LTS. If you can make the appropriate translations to something else, feel free. In fact, if you can build aarch64-linux-gnu for OS X, please do and release it to MacPorts!

---

### Required Items
* A [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/). I got mine from Newark (element14 for personal orders).
* A Micro SD card from which your software will load. The Raspberry Pi folks recommend getting an 8GB class 4 (4 MB/s) card and we *will* be loading it with Linux to start out. I actually got mine from Staples because I brought my 3B into work when it came in the mail (all excited) and forgot that I'd need one.
* A Micro SD adapter. You're going to need to be able to mount and write to Micro SD cards. My laptop has an SD slot so I used one of the SD to Micro SD adapter shells. If you're not in a rush to buy a Micro SD card at Staples due to poor planning, you can likely find a card that comes with the SD adapter. I've since switched to a USB adapter that handles a bunch of different formats. I had it lying around the house and, as it turns out, using the SD adapter required me to unseat and reseat it on my MacBook to get it to mount -- which was a pain.
* A USB to TTL serial converter. This is because our I/O will be serial-based rather than via a monitor and attached keyboard. I got a JBtek adapter from Amazon because I have Prime so it was $7 US delivered. It also provides power to the 3B which has so far been sufficient. The downside to using it for power is that you need to have a serial console that can immediately connect to the USB interface when you plug it in. I've been using Serial for that on OS X. It's $30 US. Ouch. There may have been beer involved in that purchase.
* A micro USB cable for powering the 3B from your computer or other powered USB peripheral (phone charger, etc).
* The [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/) install.

### Optional Items
* A decent micro USB power supply rather than the cable+USB-power. The 3B seems a bit picky about its power (lots of rainbow boxes fading in and out). I ended up spending $10 on the CanaKit 2.5A -- again, from Amazon -- which cleared that issue. It ultimately wasn't strictly necessary, but I was trying to sort a serial I/O issue and wanted to eliminate power as a possible problem. You might also want one if you don't care to power your 3B via the USB-TTL adapter (which I recommend against for most users).

A [more comprehensive list of requirements](https://www.raspberrypi.org/documentation/setup/) can be found at the Raspberry Pi website.

### Running Raspbian with a Serial Console
The first thing we want to do is get Raspbian installed on our Micro SD card using the [installation instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) at the Raspberry Pi website. If you have an HDMI capable monitor (HDMI is a superset of DVI, so HDMI to DVI adapters are readily available) and a USB keyboard & mouse, you can try out your newly installed image by plugging everything together and providing power via a micro USB adapter. You should see a typical Linux boot sequence followed by a desktop. If you see a rainbow looking square occasionally appear on the upper right side of the screen, that's the 3B complaining about its power quality. The only way I was able to get rid of that was via the power supply mentioned in the **Optional Items** list. That was after trying every cable and USB power source I had. I suppose I could have wired up a USB A male adapter to my bench supply, but ordering a dedicated supply seemed like a better idea. If it makes you feel any better, I noticed no ill effects from this -- it was primarily an annoyance.

If you don't have an HDMI monitor or you just want to skip straight to the serial I/O, now comes the part where we mess with the hardware. You can easily fry components hooking up the USB/TTL serial adapter. You can also fry components by trying to power your 3B from both the adapter and the micro USB port. Be careful, and take your time to get it right.

The documentation that came with my JBtek was rather sparse. So sparse, in fact, that there wasn't any. Figuring out how to connect it to the 3B took some digging around the dark corners of the internet. Okay, that's probably overly dramatic, but it wasn't trivial. Like most things, after reading datasheets and digging about, I found the bulk of what I needed via a link at the Raspberry Pi website.

The Broadcom chip in the 3B has two UARTs (Universal Asynchronous Receiver/Transmitter). One of them is used by Bluetooth so we'll be using what they call the miniUART which is mapped to the GPIO pins (the 40 pins on the edge of the board arranged in two rows of 20 pins each). Take a look at [gadgetoid's awesome Raspberry Pi pinout](http://pinout.xyz/) diagram. We'll only be using 3-pins for communications: [Pin 8 - Transmit Data (TXD)](http://pinout.xyz/pinout/pin8_gpio14), [Pin 10 - Receive Data (RXD)](http://pinout.xyz/pinout/pin10_gpio15), and a ground reference - in our case, [Pin 6 - Ground](http://pinout.xyz/pinout/ground). We could have used any of the grounds, but pin 6 is next to others which is more convenient. On my JBtek, I have 4 wires coming out the end: red, black, white, and green. For now, we're going to ignore the red wire. I plugged the black wire into ground (pin 6), the white wire into transmit (pin 8), and the green wire into receive (pin 10). If your wire colors are different you'll need to figure out which is which. Either find some documentation for your adapter, or leave a comment here and I'll see if I can help you out. The one thing to keep in mind is that the *Transmit* wire on your adapter goes to the Receive pin on the 3B, and the *Receive* pin on your adapter goes to the Transmit pin on the 3B; i.e., the names are swapped. The signals are named from the perspective of the originator, so the 3B is transmitting to something that is receiving, and *vice versa*.

At this point, I would strongly suggest leaving the red wire unhooked and powering the 3B from the micro USB port. If you absolutely are just dying to power your 3B exclusively from the USB-TTL adapter, [the instructions are here](/the-red-wire.html). You'll also do well to have a monitor attached because you can see what's going on fairly clearly.

Finally, you'll need a serial console program. Like I said, I plunked down $30 US for Serial from the Mac App store (not before using up the 7-day trial period and finding a liked it), but there are many options. I also like ```miniterm.py``` which comes with the python serial package (on Ubuntu: ```sudo apt-get install python-serial```); people who never use Ctrl-A to go to the start of a line in the shell seem to like ```screen``` (I'm not one of them); but find something that works for you. You're going to use communication parameters of 115200 baud, 8 data bits, no parity bit, 1 stop bit (115200 8 N 1). If you use miniterm, a typical startup will be ```miniterm.py /dev/ttyUSB0 115200```. If you're on Linux, you'll really want to add yourself to the dialout group so you don't have to use ```sudo``` to launch miniterm, screen, cu, or whatever. I usually just edit the /etc/group file as root and add my username after the colon at the end of the line that starts with ```dialout```, then log out and back in again. Or you can ```sudo adduser <your user name> dialout``` and then logout and back in again.

Now we're ready to try out our serial console. Plug in your USB-TTL adapter and connect to it via whatever serial console you decided to use. Once you've done that, apply power via the micro USB cable and watch your glorious serial data come pouring out to your console (for those that are powering via "the red wire", plug in the USB-TTL adapter **only** and then connect your console - if you're quick about it, you should be to catch some of the data coming out).

What's that? You only see a garbled mess? Yeah. Me too. Tracking down this problem was one of the reasons I bought the CanaKit power supply -- in order to rule out bad power as part of the problem. I also bought a different brand of USB-TTL adapter. It's a shame I don't have a digital capture scope that can do protocol analysis. Wait -- I do. >_>

As it turns out, the miniUART is clocked via the Broadcom's core clock and running at default frequency (400MHz) it can't divide its baud rate to 115200. We need to lower it to 250MHz to see serial output correctly. So power off your 3B, mount your micro SD card on your host machine, and add the following line to the config.txt file in the boot partition (the boot partition is a 60MB FAT32 formatted partition):

```core_freq=250```

If you are running your 3B with a monitor+keyboard+mouse, you can actually do it from there by opening the drive that mounts as "boot" and editing it in place. A lot of the tutorials I see recommend that you back up everything in your boot partiiton before you touch config.txt since borking it up can keep your 3B from booting, but we're starting from a scratch system so absolute worst case is that you just reinstall Raspbian on your micro SD card and try again.

Okay. Unmount your micro SD card, put back in your 3B, and repeat the boot steps from above. **Now** you should have legible serial output. You can login if you want. The user is **pi** and the password is **raspberry**.

That wraps up **Getting Started**. Next time we'll setup our Ubuntu VM (if necessary) and get some basic serial I/O working. Feel free to ask questions or leave feedback via the comments section below.

### Troubleshooting
1. Getting absolutely no serial output.
    1. Is your communication software set up for 115200,8,N,1?
    2. Is your ground wire connected?
    3. Are your transmit and receive wires connected?
    4. You probably have your transmit and receive wires swapped. They're 3.3V nominal, low current, and should be high impedance (resist short circuits). It's highly unlikely you've caused any damage to anything by doing this.