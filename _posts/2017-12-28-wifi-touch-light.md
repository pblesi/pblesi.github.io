---
layout: post
title: Building a Wifi Touch Light
date: '2017-12-28T19:45:00.001-06:00'
author: Patrick Blesi
related_posts:
  - particle-manual-wifi-creds
  - manual-switch-touch-lights
tags:
- DIY
- Arduino
- Particle Photon
- Electronics
modified_time: '2017-12-28T19:45:00.001-06:00'
---

<p>
This fall a recent death in the family left a bit of a gap in our life. My wife started searching for a way for our family to better keep in contact with each other. After some searching, she came across these [long distance touch lamps](https://www.uncommongoods.com/product/long-distance-touch-lamp?country=US&aw_cid=418702497&aw_aid=23036126097&aw_dev=c&aw_loc=9021727&aw_key=&aw_mtype=&aw_net=g&aw_ad=89994005337&aw_pos=1o1&aw_shopid=43038&aw_prod_partid=104832534101&gclid=Cj0KCQiAjO_QBRC4ARIsAD2FsXNdhbcVkq8L4WTLYsobNu_gqrBxJ0mkMVUMwLcnoM7Z61gX2-oWWOYaAsXZEALw_wcB). Intrigued by the concept, I searched to see if I could build a similar product at a more appealing price point.
</p>

<p>
  <img class="img" src="/assets/img/wifi_touch_lights_01.png" alt="Wifi Touch Lamps" width="750">
</p>

<p>
After some searching, I discovered the [kickstarter project](https://www.kickstarter.com/projects/johnharrison/filimin-a-wi-fi-enabled-touch-light-that-connects) of the same lamps found on Uncommon Goods and the creator's step-by-step how-to on [Instructables](http://www.instructables.com/id/Networked-RGB-Wi-Fi-Decorative-Touch-Lights/) explaining how he made the Filimin touch lamp. What follows is my own how-to on how I created wifi enabled touch lamps to connect my wife, her mom, grandma, and aunt.
</p>

<p>
The wifi lamps change color when touched and synchronize their colors to be the same no matter where in the world they are located. Each lamp gets assigned a default color so you know who has touched their lamp. This implementation is adapted and heavily borrowed from John Harrison's original touch lights.
</p>

## Materials, Supplies, And Tools

Below are the materials, supplies, and tools I used to make my touch lamps.

### Materials

* [Unifun Touch Lamp](https://amzn.to/2DxjlnQ) ($18)
* [Particle Photon](https://www.adafruit.com/product/2722) without headers ($19)
* [NeoPixel Ring 24](https://www.adafruit.com/product/1586) ($17)
* [10M-Ohm Resistor](https://www.radioshack.com/products/radioshack-10m-ohm-1-4-watt-carbon-film-resistor-5-pack) ($1)
* [3/8" felt pads](https://www.lowes.com/pd/Waxman-84-Pack-Brown-Round-Felt-Pad/3034718?cm_mmc=SCE_PLA-_-ToolsAndHardware-_-FloorProtection-_-3034718:Waxman&CAWELAID=&kpid=3034718&CAGPSPN=pla&store_code=1845&k_clickID=69a86dd4-e813-4697-8b04-8fbf57efb339&gclid=CjwKCAiA7JfSBRBrEiwA1DWSG7ADxYMwlrxaN9Mj14AvM7i9ufVifcWPjilJUcnCNGscnj-vTHxgqhoCuNgQAvD_BwE) for covering screw holes on the bottom of the lamp
* A USB phone charger, should be name brand such as Apple, Samsung, Motorola, etc.
* A wooden dowel (1-1/2" inch) for bracing the Photon when plugging in the USB cable

### Supplies

* Wire (gauge 22AWG)
* Solder
* Hot Glue
* Goo Gone to remove sticky adhesive from the bottom of the lamp

### Tools

* Screwdriver
* Razorblade
* Soldering iron
* Hot glue gun
* Drill and 3/32" drill bit
* Wirecutter/wire stripper

## Step 1: Disassemble the Unifun Touch Lamp

The first step is to disassemble your Unifun Touch Lamp. The finished product should look like the following.

  <p>
    <img class="img" src="/assets/img/wifi_touch_lights_02_01.png" alt="Unifun Touch Lamp packaging" width="375">
    <img class="img" src="/assets/img/wifi_touch_lights_02_02.png" alt="Disassembled wifi touch lamps" width="375">
  </p>

1. Remove the foam ring from the bottom of the touch lamp. Be careful to remove as much of the stickiness as possible from the bottom of the lamp.
1. Unscrew the screws from the bottom of the lamp and remove the bottom plate.
1. Remove the four silver screws to loosen the top white dome from the base of the lamp. Do not remove the dome yet.
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_03.png" alt="Unifun Touch Lamp packaging" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_04.png" alt="Unifun Touch Lamp contents" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_05.png" alt="Foam base ring on Unifun Touch Lamp" width="200">
    </p>
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_06.png" alt="Sticky residue under foam ring" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_07.png" alt="Silver screws connecting white dome to the lamp base" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_08.png" alt="Loosened white dome" width="200">
    </p>
1. Peer between the white dome and the base for a thin wire connecting the top touch plate to the circuitboard. Cut this wire with a razorblade or wire cutter.
1. Unscrew and remove the circuit board from the lamp base.
1. Use a screwdriver to push the flaps holding the touch plate to the white dome.
1. Use a soldering iron to remove the wire connected to the touch plate.
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_09.png" alt="Unifun lamp circuit board" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_10.png" alt="Inside of lamp white dome" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_11.png" alt="Disassembled Unifun Touch Lamp" width="200">
    </p>

## Step 2: Prepare the Shell

Once you have your lamp disassembled, you need to prepare it to be reassembled with your wifi-enabled components.

1. Use Goo Gone or some other solvent to remove the sticky residue off of the bottom plate of the lamp.
1. Use a 3/32" drill bit to drill a hole through the center of the battery compartment of the white dome.
1. Cut a 6-3/4" piece of wire and solder it to the touch plate.

<p>
  <img class="img" src="/assets/img/wifi_touch_lights_12.png" alt="Hole drilled in white dome" width="375">
  <img class="img" src="/assets/img/wifi_touch_lights_13.png" alt="Wire soldered to touch plate" width="375">
</p>

## Step 3: Connect the Electrical Components

Now it's time to connect the electrical components together.

1. Solder a 3-3/4" wire to each of the VIN, GND, and D2 pins of the Particle Photon.
1. Solder the 10M-Ohm resistor to pins D3 and D4 of the Photon.
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_14.png" alt="Touch lamp electrical components" width="350">
      <img class="img" src="/assets/img/wifi_touch_lights_15.png" alt="Photon with components soldered on" width="350">
    </p>
1. Thread the wires through the holes in the base and solder them to the NeoPixel. VIN should connect to "PWR +5V", GND should connect to "GND", and D2 should connect to "Data Input". There may be multiple power and ground pins, choose the ones that are closest to the wires. It is helpful to plug a USB cable (unpowered) into the photon through the USB hole in the lamp base to help hold the Photon in place.
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_16.png" alt="Wires thread through lamp base" width="350">
      <img class="img" src="/assets/img/wifi_touch_lights_17.png" alt="NeoPixel soldered to the Photon" width="350">
    </p>

## Step 4: Test the Lamp's Functionality

At this point we are ready to test the Lamp's functionality to ensure we soldered everything correctly and all the components are working.

If you have not yet set up your Photon, plug it in and use the app provided by Particle to connect to your Photon and get it connected to the internet via wifi. Particle provides good [instructions](https://docs.particle.io/guide/getting-started/start/photon/) for getting your Photon connected.

Once you have your photon connected, you should set up your development environment for getting code onto the Photon, I prefer using their [command line tool](https://docs.particle.io/guide/tools-and-features/cli/photon/), but their [web IDE](https://docs.particle.io/guide/getting-started/build/photon/#logging-in) is pretty nice as well if you want to get started quickly.

If you are using the command line tool, then you can clone the [repository](https://github.com/pblesi/touch_light) containing the code from Github in order to flash it onto the Photon, otherwise you will need to copy and paste the code into the web browser in order to use it. I will assume you are doing the former.

1. Once you have connected your Photon to the internet, installed the command line tool, and logged in, you can clone the touch_light repo by running:

        $ git clone git@github.com:pblesi/touch_light.git

1. In `src/touch_light.ino` update `NUM_PARTICLES` with the number of touch lamps you intend to have linked together.
1. Update `particleId[]` with the 24 character unique identifier for your Photon. This can be found by running `particle list` on the command line.
1. Update `particleColors[]` with your prefered default color for this lamp. Some trial and error may be required to identify which color corresponds to which number. Each color is a value between 0 and 255 that maps onto the [RGB Color wheel](https://en.wikipedia.org/wiki/Color_wheel#/media/File:RBG_color_wheel.svg). 0 and 255 correspond to green.
1. Compile your code into a firmware `.bin` file.

        $ particle compile photon

1. Flash your photon

        $ particle flash <the name of your touch lamp> <the generated firmware.bin file>

1. You should see a white flash, a cycle through the color wheel, and then a fade out on green. If you do not see this behavior, check for a short in the circuits. You should not touch the lamp between when the light flashes and when it traverses the color wheel as it is calibrating its touch sensor.

## Step 5: Secure the Components

Once you have verified the lamp lights up as expected, you can secure the electrical components to the lamp fixture.

1. Apply four dollops of hot glue to the lamp base in order to secure the NeoPixel. Do this quickly to prevent having the glue prematurely dry without securing the NeoPixel.
1. Secure the wooden dowel to the lamp base (I used a pencil) behind the Photon board. This is used to prevent the Photon from moving when a USB cable is plugged into it. When gluing the wooden dowel, be careful to not cover the three holes that are used to secure the white dome to the lamp base (the fourth hole will be covered by the Photon).
1. Unplug the USB cable from the Photon and check if it comes loose. If the Photon comes loose, secure it by gluing it to the base. If you cannot remove the Photon, put glue around the edges to make sure it stays in place. As soon as the Photon is put in place, make sure the Photon is aligned with the USB hole and plug the USB cable into the Photon to ensure it fits easily when the Photon is secured.
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_18.png" alt="NeoPixel glued to the lamp base" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_19.png" alt="Wooden dowel glued to secure the Photon" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_20.png" alt="Photon glued to the lamp base" width="200">
    </p>

## Step 6: Reassemble the Lamp

You're almost finished! All that's left is to connect the touch plate to the Photon and reassemble the lamp.

1. Screw the white dome back into the lamp base.
1. Reattach the touch plate to the white dome, threading the attached wire through the hole drilled in the battery compartment.
1. Solder the wire from the touch plate to the D3 side of the resistor.
1. Screw the bottom plate back on to the base.
1. Place six felt pads along the bottom ring of the lamp, covering the exposed screw holes.
1. You're done! You can now play with your wifi touch lamp!
    <p>
      <img class="img" src="/assets/img/wifi_touch_lights_21.png" alt="Touch plate soldered to the Photon" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_22.png" alt="Exposed screw holes on the touch lamp" width="200">
      <img class="img" src="/assets/img/wifi_touch_lights_23.png" alt="Felt pads covering the screw holes" width="200">
    </p>

## Final Product

I built four lamps to give to my wife, her mother, grandma, and aunt. They work beautifully and were a joy to build. As a first Photon project, they were easy to get started and assemble. Check out the video below for a demonstration of how they work.

<iframe width="750" height="422" src="https://www.youtube.com/embed/DX0LaWrvIqY?rel=0" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>
