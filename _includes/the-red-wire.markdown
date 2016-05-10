
### The Red Wire

If you insist on ignoring my advice, then go ahead and attach the red wire (or whatever your +5V wire is) to [Pin 2 - 5v Power](http://pinout.xyz/pinout/pin2_5v_power). You could have used pin 4, but keeping power and ground separated a bit helps reduce the odds of accidental contact between them.

---

# CAUTION! DO **NOT** CONNECT BOTH THE SERIAL ADAPTER'S 5V POWER (RED WIRE) AND THE MICROUSB POWER CABLE IN AT THE SAME TIME! YOU CAN SHORT OUT YOUR SYSTEM AND DAMAGE IT!

### Obviously they both have to also be connected to power at the same time for this to happen, but all the same don't have them both plugged into the board; it's really easy to lose track of what you're doing and apply power to both simultaneously.

---

So here's the chicken and the egg part: in order to have the /dev/ttyUSB0 (or whatever it shows up as) device available to your serial console software, you'll need to plug in the USB-TTL adapter. But if you're also powering the 3B from the adapter, you'll miss out on the startup info coming out of the serial port; hence my recommendation that you not power from the USB-TTL adapter. Being one to not follow my own advice, I actually do power from the USB-TTL adapter but I also use the Serial program on OS X which can handle the USB device disappearing and reappearing. Did I mention it was expensive? And that there may have been beer involved?