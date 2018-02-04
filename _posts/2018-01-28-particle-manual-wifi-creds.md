---
layout: post
title: Manually Setting Particle Wifi Creds
date: '2018-01-28T19:45:00.001-06:00'
author: Patrick Blesi
related_posts:
  - wifi-touch-light
tags:
  - DIY
  - Particle Photon
  - Electronics
  - Maker
  - IOT
modified_time: '2018-01-28T19:45:00.001-06:00'
---

<p>
I recently created a number of [wifi touch lights]({{ site.baseurl }}{% post_url 2017-12-28-wifi-touch-light %}) to help my family keep in "touch" with each other. It was a fun project, but there was a bit of a challenge getting everyone's light set up. I was giving the lights as gifts, and I did not want my family members (some of them not as tech savvy) to have to go through the Photon wifi setup using a [web application](https://docs.particle.io/guide/getting-started/start/photon/#step-2a-connect-your-photon-to-the-internet-using-the-setup-web-application) or [mobile app](https://docs.particle.io/guide/getting-started/start/photon/#step-2b-connect-your-photon-to-the-internet-using-your-smartphone).
</p>

<p>
Luckily, Particle's API provides a way to set wifi credentials for your Photon from within the source code of the flashed program. They provide great documentation describing this [API](https://docs.particle.io/reference/firmware/photon/#wifi), however they don't provide any tutorials for the steps necessary to set wifi credentials from within your code. Below I describe how to set up wifi credentials manually for your Particle Photon.
</p>

<p>
The first step is to place your photon in [SEMI_AUTOMATIC](https://docs.particle.io/reference/firmware/photon/#semi-automatic-mode) mode. This must be done outside of the `setup` and `loop` functions.
</p>

```c++
SYSTEM_MODE(SEMI_AUTOMATIC);

setup() {
  // Setup code goes here
}

void loop() {
  // Do more stuff here
}

```

<p>
`SEMI_AUTOMATIC` mode does not immediately connect to the particle cloud, but waits until you manually call `Particle.connect()` to connect to the cloud. Once connected to the cloud, your particle automatically manages the connection.
</p>

<p>
Once your Photon is put into `SEMI_AUTOMATIC` mode, you can manually set up your wifi connection at the beginning of your `setup` function. You can specify up to five sets of credentials for Photons and seven sets of credentials for Cores. If more credentials are specified, the oldest credentials will be overwritten.
</p>

```c++
SYSTEM_MODE(SEMI_AUTOMATIC);

void setup() {
  WiFi.on();
  WiFi.disconnect();
  WiFi.clearCredentials();
  WiFi.setCredentials("SSID", "password");
  WiFi.setCredentials("ALTERNATE_SSID", "other_password");
  WiFi.connect();
  waitUntil(WiFi.ready);
  Particle.connect();

  // Rest of setup code goes here
}

void loop() {
  // Do more stuff here
}
```

<p>
You can clean this code up a bit by placing the wifi setup code in its own function:
</p>

```c++
SYSTEM_MODE(SEMI_AUTOMATIC);

setup() {
  setupWifi();

  // Rest of setup code goes here
}

void loop() {
  // Do more stuff here
}

void setupWifi() {
  WiFi.on();
  WiFi.disconnect();
  WiFi.clearCredentials();
  WiFi.setCredentials("SSID", "password");
  WiFi.setCredentials("ALTERNATE_SSID", "other_password");
  WiFi.connect();
  waitUntil(WiFi.ready);
  Particle.connect();
}
```

<p>
This is a pretty good solution, but if you commit your code to a source code management system like Git (which you should), then this makes it hard to commit your code without commiting your wifi credentials. This can be remedied by placing our credentials in a separate file:
</p>

```c++
/*
 * src/wifi_creds.h
 */

const String wifiCredentials[][2] = {
  // Set wifi creds here (last entry will be tried first when connecting)
  // {"SSID", "password"}
};
```

```c++
/*
 * src/main.ino
 */

#include "wifi_creds.h"

SYSTEM_MODE(SEMI_AUTOMATIC);

setup() {
  setupWifi();

  // Rest of setup code goes here
}

void loop() {
  // Do more stuff here
}

void setupWifi() {
  WiFi.on();
  WiFi.disconnect();
  WiFi.clearCredentials();
  int numWifiCredentials = sizeof(wifiCredentials) / sizeof(*wifiCredentials);
  for (int i = 0; i < numWifiCredentials; i++) {
    WiFi.setCredentials(wifiCredentials[i][0], wifiCredentials[i][1]);
  }
  WiFi.connect();
  waitUntil(WiFi.ready);
  Particle.connect();
}
```

<p>
You can commit `src/wifi_creds.h` to be used as a template, and then not commit this file when you have added credentials.
</p>

<p>
What if you want to flash firmware onto your Photon, but don't want to overwrite the wifi credentials currently on your Photon? You can make one more tweak to your code to allow for updating the Photon's firmware without manually specifying wifi credentials:
</p>


```c++
/*
 * src/wifi_creds.h
 */

// Uncomment the line below if specifying credentials in this file
// #define WIFI_CREDENTIALS_SPECIFIED

const String wifiCredentials[][2] = {
  // Set wifi creds here (last entry will be tried first when connecting)
  // {"SSID", "password"}
};
```

```c++
/*
 * src/main.ino
 */

#include "wifi_creds.h"

#ifdef WIFI_CREDENTIALS_SPECIFIED
SYSTEM_MODE(SEMI_AUTOMATIC);
#endif

setup() {
#ifdef WIFI_CREDENTIALS_SPECIFIED
  setupWifi();
#endif

  // Rest of setup code goes here
}

void loop() {
  // Do more stuff here
}

void setupWifi() {
  WiFi.on();
  WiFi.disconnect();
  WiFi.clearCredentials();
  int numWifiCredentials = sizeof(wifiCredentials) / sizeof(*wifiCredentials);
  for (int i = 0; i < numWifiCredentials; i++) {
    WiFi.setCredentials(wifiCredentials[i][0], wifiCredentials[i][1]);
  }
  WiFi.connect();
  waitUntil(WiFi.ready);
  Particle.connect();
}
```

<p>
By commenting or uncommenting `#define WIFI_CREDENTIALS_SPECIFIED` you can control if code-specified credentials are placed on the Photon.
</p>

<p>
Setting up particle projects in this way will give you the flexibility to set your Photon's wifi credentials manually, without having to use additional tools such as Particle's mobile or web applications.
</p>

<p>
Cheers!
</p>
