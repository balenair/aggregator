# aggregator
Aggregate air quality readings from multiple balenair devices on a dynamically-generated dashboard.

![sample dashboard](https://raw.githubusercontent.com/balenair/aggregator/main/aqi-summary-4.PNG)

## Setup

This project is designed to run on an x86 device using [balenaCloud](https://www.balena.io/cloud), but can easily be modified for use on an Arm device such as a Raspberry Pi. It is easiest to set up when running on the same network as the Balenair devices, but can be deployed anywhere with some slight configuration changes.

Setting up the software is as easy as clicking the button below:

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/balenair/aggregator)

Alternatively, you can clone this repo to your development machine and use the [balenaCLI](https://docs.balena.io/reference/balena-cli/) to push the code to your devices.

Once you have your Aggregator running, you need to configure your Balenair devices to publish their AQI to it. Either using a fleet variable or per-device, add the `AGG_ADDRESS` variable to your Balenair with a value of the Aggregator's IP address. This will be straightforward when the Aggregator is on the same network as the Balenair devices. If the Aggregator is on a different network, you'll need to make sure it is publicly reachable. This may entail using port-forwarding or hosting the device on a cloud server such as an AWS AMI. If you place the Aggregator on a different network, see the security section below.

## How it works

The aggregator is built out of [balena Blocks](https://github.com/balenablocks) which are drop-in pieces of functionality. The MQTT service receives data from the Balenair devices which is picked up by the [Connector block](https://github.com/balena-labs-projects/connector) and stored in a local InfluxDB database. The [Dashboard block](https://github.com/balena-labs-projects/dashboard) sees this data and pulls it into the defualt dashboard. This simple dashboard will list any device's air quality index that is detected, and since it groups on the topic name (which includes the device name) it will dynamically add or delete devices as needed. Note that the Dashboard block is only available prebuilt for armv7 so we build it here manually with a few slight modifications.

## Security considerations

Out of the box, the Aggregator has minimal security and is designed to run on an internal network where it is not exposed directly to the internet. (i.e. behind a router or firewall) If you will be opening it up to the internet directly by port forwarding or hosting in a datacenter, the following should be implemented:

 - Add usernames and passwords to the Aggregator's Mosquitto broker and configure it so that clients must authenticate (You can use the Balenair's `AGG_USERNAME` and `AGG_PASSWORD` variables) - there's more information in the [configuration man page](https://mosquitto.org/man/mosquitto-conf-5.html).
 
  - Enable some sort of secure communications between the client and the Aggregator. By default, MQTT is unencrypted and can be intercepted by third parties. A combination of TLS and/or payload encryption should be considered. See [this article](http://www.steves-internet-guide.com/mqtt-security-mechanisms/) for some suggestions.
  
  - Use access control in Mosquitto to limit topics clients can read and write to. See [this article](https://jurian.slui.mn/posts/smqttt-or-secure-mqtt-over-traefik/) for an example.
  
  - Add a username and password to the Aggregator dashboard by using the "sign in" button on the lower left of the main page. The default username and password are `admin`.
  
## Additional features	

This project is a work in progress, and currently has limited functionality. Feel free to add suggestions, comments or enhancement ideas in the issues section, and as always, PRs are welcome!
