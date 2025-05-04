---
title: "Mobilematic 1.0.0 Release 🎉"
date: 2025-04-09 21:38:00 +0200
categories: [Mobilematic, Release]
tags: [mobilematic, app, flutter, mobile, raspberrymatic, homematic]     # TAG names should always be lowercase
description: "This post is about my first App: Mobilematic"
---
## TL;DR
I released an app to the google playstore which can control smart home devices from homematic using raspberrymatic. For development I used Flutter and the current main features include controlling thermostats, shutters and simple management of programs. The most important things I learned are:
- Designing a sustainable architecture for efficient maintenance
- State mangement using Flutter
- Securely store credentials on a mobile (android) device
- Release an app to the google playstore

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
One of the first challenges i faced, was how to efficiently handle the addition of supported devices. Using the XML-API each device has specific attributes. The data for a device for example looks like this:
```xml
<device name='Thermostat Living Room' ise_id='1643' unreach='false' config_pending='false'>
	<channel name='Thermostat Living Room' ise_id='1644' index='0' visible='true' operate='true'>
		<datapoint name='HmIP-RF.00399F299C4BFB:0.CONFIG_PENDING' type='CONFIG_PENDING' ise_id='1645' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.LOW_BAT' type='LOW_BAT' ise_id='1651' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.UNREACH' type='UNREACH' ise_id='1659' value='false' valuetype='2' valueunit='' timestamp='1668025987' operations='5' />
		<datapoint name='HmIP-RF.00399F299C4BFB:0.UPDATE_PENDING' type='UPDATE_PENDING' ise_id='1663' value='false' valuetype='2' valueunit='' timestamp='1666901046' operations='5' />
        <!-- ... more datapoints ... -->
	</channel>
	<channel name='Thermostat Control Living Room' ise_id='1667' index='1' visible='true' operate='true'>
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
In the example the device has attributes like `ACTUAL_TEMPERATURE` and `SET_POINT_TEMPERATURE` which represent the measured temperature and the configured temperature. These attributes can vary depending on the device hence the challenge is to find a way to efficiently manage supported devices without hardcoding each attribute. The solution I came up with, is a mapping file which I call Driver. Each type of a device has usually the same functionalities. For example a thermostat has the functionality to read and set the temperature. So in the file I define the mapping of the actual attributes from the device to custom properties which I use in the code.

When I need to add a new device and the device type is already supported, just a single file needs to be added which maps the attribute types from the device to my custom ones which I use in the code. Of course when a completely new device type needs to be added, then also the new UI and functionality has to be added.

### State management
Another challenge was the state management. Initially I used the `setState` method from Flutter which notifies the framework that the internal state of the object has changed and causes the framework to schedule a build for the object. I also used `ChangeNotifier` to propagate the change of a value. By the time the code got messy and was somewhat hard to test.

I discoverd [flutter_bloc](https://pub.dev/packages/flutter_bloc) which is a library for state management. There might be other solutions, but bloc sounded interesting so I decided to learn it and integrate it into the app. I replaced my custom solution with bloc, and the code got cleaner and also better testable.

### Security
One thing I identified as important for the release, was security. Since the App just acts as a client which connects to the CCU, there aren't many security aspects. One important aspect though is the storage of the credentials on the mobile device.
To solve this problem, I used [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage).
With this library you can store AES encrypted key-value pairs using SharedPreferences. The key is stored using the platforms integrated mechanism, like Keychain for IOS and KeyStore for android.

## Release process
The release process was pretty annoying. Since I'm doing this as a hobby and don't have a company for that, I created a so called personal developer account. Releasing an app with such an account requires you to find at least 12 people, who must be "opted-in" for at least 14 consecutive days. It is nowhere documented, what "opted-in" exactly means. Like do they need to open the app everyday? Do they need to interact with it?
Apperantly this just means, to have the app installed. That's all.

I also had to create a website containing the privacy policy for the app. Luckily there isn't much for my app. I used Github Pages for that which I would really recommend, since it's fast, easy and free.

Besides that I had to configure the standard stuff like descriptions, answer some questions about my app, upload screenshots etc. After some time and reviews from Google everything was okay I could finally release it to production 🥳.