---
title: "Mobilematic 1.0.0 Release 🎉"
date: 2025-04-09 21:38:00 +0200
categories: [Mobilematic, Release]
tags: [mobilematic, app, flutter, mobile, raspberrymatic, homematic]     # TAG names should always be lowercase
description: "This post is about my first App: Mobilematic"
---
## TL;DR
I released an app to the google playstore which can control smart home devices from homematic using raspberrymatic. For development i used Flutter and the current main features include controlling thermostats, shutters and simple management of programs. The most important things i learned are designing a sustainable architecture for efficient maintenance, state mangement in Flutter, how to securly store credentials on an android device and how to release an app to the playstore.

## Story
A friend of mine had smart home devices from Homematic. As a CCU (Central Control Unit), he used a Raspberry Pi running RaspberryMatic to manage everything. At the time, he was already using an app to control the devices, but there were a few things about it that bothered him. We talked about it, and half-jokingly, he asked if I could build a better app. That sparked my curiosity, and just for fun, I started hacking something together. I quickly realized how much I enjoyed it and gradually got more interested in the topic. Eventually, I even bought my own smart home devices so I could use the app myself. Over time, I kept adding features and maintaining it.

After putting a lot of time into the app—and using it almost daily, especially during winter—I thought it would be awesome if others could download and use it too. So I defined a target feature set and a list of bugs that needed fixing before it would be ready for the Play Store. Once everything was in place, I polished and prepared the app for release. After pushing through the (annoying but mandatory) closed testing phase, the app was finally published on March 30, 2025 🔥.

## Technical stuff
### Key data

| Category                      | Answer          | Comment |
| :--------------------------- | :--------------- | :------ |
| Target Platform                   | Android     | My friend and i had both android devices |
| Technology                      | Flutter/Dart    |      I had already used Flutter for university projects and found it very useful for building apps which doesn't use native features |
| RaspberryMatic API | XML-API |   I started with it, because it was very simple and had all the functionality i needed. https://www.homematic-inside.de/software/xml-api |
| Design | Figma |   Figma is very a very powerfull and free tool, which is great for example for creating simple icons |

The first couple of commits were very hacky and just about playing around with the api. But soon i noticed that if i want to continue this app i need to think about some architecture to keep the code maintainable.

### Add suport for new devices
One of the first challenges i faced, was how to efficiently handle the addition of supported devices. Using the XML-API each device has specific attributes. For example like this:
```xml
<device name='Heizkörperthermostat Wohnzimmer' ise_id='1643' unreach='false' config_pending='false'>
	<channel name='Heizkörperthermostat Wohnzimmer' ise_id='1644' index='0' visible='true' operate='true'>
		<datapoint name='HmIP-RF.00399F299C4BFB:0.CONFIG_PENDING' type='CONFIG_PENDING' ise_id='1645' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.LOW_BAT' type='LOW_BAT' ise_id='1651' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.UNREACH' type='UNREACH' ise_id='1659' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.UPDATE_PENDING' type='UPDATE_PENDING' ise_id='1663' value='false' valuetype='2' valueunit='' timestamp='1666901046' operations='5' />
        <!-- ... more datapoints ... -->
	</channel>
	<channel name='Heizkörperthermostat Steuerung Wohnzimmer' ise_id='1667' index='1' visible='true' operate='true'>
		<datapoint name='HmIP-RF.00399F299C4BFB:1.ACTIVE_PROFILE' type='ACTIVE_PROFILE' ise_id='1668' value='1' valuetype='16' valueunit='' timestamp='1668025987' operations='7' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.ACTUAL_TEMPERATURE' type='ACTUAL_TEMPERATURE' ise_id='1669' value='22.600000' valuetype='4' valueunit='°C' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.ACTUAL_TEMPERATURE_STATUS' type='ACTUAL_TEMPERATURE_STATUS' ise_id='1670' value='0' valuetype='16' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.BOOST_MODE' type='BOOST_MODE' ise_id='1671' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='6' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.CONTROL_MODE' type='CONTROL_MODE' ise_id='1674' value='' valuetype='16' valueunit='' timestamp='0' operations='2' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.SET_POINT_MODE' type='SET_POINT_MODE' ise_id='1685' value='0' valuetype='16' valueunit='' timestamp='1668025987' operations='7' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.SET_POINT_TEMPERATURE' type='SET_POINT_TEMPERATURE' ise_id='1686' value='21.000000' valuetype='4' valueunit='°C' timestamp='1668025987' operations='7' />
		<datapoint name='HmIP-RF.00399F299C4BFB:1.WINDOW_STATE' type='WINDOW_STATE' ise_id='1690' value='0' valuetype='16' valueunit='' timestamp='1668025987' operations='7' />
        <!-- ... more datapoints ... -->
	</channel>
	<channel name='HmIP-eTRV-B-2 R4M 00399F299C4BFB:2' ise_id='1691' index='2' visible='true' operate='true'></channel>
	<!-- ... more channels ... -->
</device>
```
The device is a radiator thermostat which has attributes like `ACTUAL_TEMPERATURE` and `SET_POINT_TEMPERATURE` which represent the measured temperature and the configured temperature. These attributes can vary depending on the device hence the challenge is to find a way of efficiently manage supported devices without hardcoding each attribute. The solution i came up with, is a mapping file which i call Driver. Each type of the device has usually the same functionalities. For example a thermostat has the functionality to read and set the temperature. So in the mapping file i define the mapping of the actual attributes from the device to a custom property which i actually use in the code. For each supported device type i define custom properties to handle the specific functionality.

When i need to add a new device and the device type is already supported, i just need to add a single file which maps the attribute types from the device to my custom ones which i use in the code. Of course when i add a completely new device type, i also need to add the UI and functionality to make it work.

### State management

### Security

## Release process

