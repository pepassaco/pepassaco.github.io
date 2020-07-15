---
layout:  post
title: "RF harvesting: an attempt to power devices through electromagnetic radiation"
author: "Pablo Álvarez"
categories: antennas
tags: [
  electronics
]
image: /assets/img/posts/2019-04-12-RFHarvesting/3.jpg

---

This week, I spent some time investingating about energy harvesting: the art of finding an autonomous way to power an electronic device - not chargers nor battery replacements. Nowadays, there are lots of techniques for energy harvesting (solar power, piezoelectricity, rf harvesting, thermoelectricity). However, it seems that, with the current state of the art, only photovoltaic energy is able to power devices with a relatively moderate power consumption; for the rest of technologies, only extremely low pwoer devices can be powered.

I wanted to give a shot to one of these alternative ways of harvesting energy, and this is how I ended up trying *RF harvesting*.

## How it works

Right now, there are lots of electromagnetic waves traveling throught the air around you. These waves carry certain ammount of energy, which, if we could receive in some way, we could use it to power our devices.

In order to receive these waves, we must use an antenna. However, the frequencies we will try to receive will vary a lot depending on if they're signals from FM bradcasting, cell phone communications, AM broadcasting, DVB-T, etc. Then, we would need some sort of broadband antenna. Another option would be focusing on a specific signal, such as FM or AM radio due to their great power, and tuning the antenna for those frequencies. In any case, for the testing purposes of this experiment, we will use a simple long-wire antenna (it would be interesting to try some others such as a Vivaldi antenna in a future).

The signal received at the endpoints of the antenna will have the form of AC current. In order to power an electronic device with this energy, we must first convert it to DC. This is why these devices are sometimes also known as *rectennas*. Since we are not working with extremely high frequencies, we should use some diodes for this purpose. A good configuration to start with would be a full bridge rectifier:

![Schematic](/assets/img/posts/2019-04-12-RFHarvesting/3.jpg)

Note that real life diodes will have a polarising voltage $V_\gamma>0$. This value is normally around 0.7 for common silicon diodes. This means that, in order to generate some DC current at the output of the rectifyer, we would need (in case of using a regultar full bridge rectifyer) an input current of around 1,4V! This is of corse incredibly high compared to the voltage we will have at the output of our antenna. In order to try to solve this issue, we can try the following:

  - **Use Germanium or Schottky diodes**: These diodes have a polarization voltage of around 0,3V, less than half the one obtained with silicon diodes.
  - **Use a transformer or voltage doubler**: Unfortunately, I will not cover these aspects in this posts since I lacked the necessary material to try them. Nevertheless, it would be nice to investigate a bit deeper on this in the future.
  - **Replacing the full bridge rectifyer with another topology**: This is a must for advanced RF Harvesting. Yet, we will use it as this project is just for demonstration purposes.

Lastly, we will need some component where we can store the energy we are receiving in order to ration it and liberate the ammount the load needs every instant. In this case, we will use the capacitors that form the last step of the recifyer. Increasing their capacitance allows us to use them as a energy storage device. I decided yo use some 220uF capacitors.

## Tests

This is the final build:

![Build](/assets/img/posts/2019-04-12-RFHarvesting/1.jpg)

This design was not able to charge the capacitors with normal ambient radiation. However, if I did not connect any load to them and measure their voltage with a multimeter, it was possible to notice how it increased when the converter was placed close to the WiFi router. 

As this was kind of a proof-of-concept, I decided to transmit some relatively powerful signals in order to see whether the harvested was able to work propperly under certain circumstances. For that, I hooked up a digital watch to its output, and used a baofeng to transmit some audio in the VHF Ham band.

Surprisingly, the watch powered on! Futhermore, the screen went completely black after I tried the High-power option on the Baofeng. The watch screen kept for about 30 seconds after I finished holding the PTT button on the transmitter, which means that its power consumption is clearly greater than the energy it is able to capure from the environment.

![Power](/assets/img/posts/2019-04-12-RFHarvesting/2.jpg)

## Future improvement

I really enjoyed this project and would like to improve it in the future: trying lower polarization voltage diodes voltage doublers, a tuned antenna or an impedance matching network could improve a lot the harvesting capabilities of the device. Stay tuned for an hypothetical new entry!
