---
title: "Mobilematic 1.0.0 Release 🎉"
date: 2025-05-04 19:21:00 +0200
categories: [Mobilematic, Release]
tags: [mobilematic, app, flutter, mobile, raspberrymatic, homematic]
description: "This post is about my first app: Mobilematic"
---

## TL;DR

I released an app to the Google Play Store that can control smart home devices from Homematic using RaspberryMatic. I developed it using Flutter. The main features include controlling thermostats, shutters, and basic program management. Here are the key things I learned:

- Designing a sustainable architecture for maintainability
- State management in Flutter
- Securely storing credentials on Android devices
- Releasing an app to the Google Play Store

---

## Story

A friend of mine had smart home devices from Homematic. He used a Raspberry Pi running RaspberryMatic as the central control unit (CCU). At the time, he was already using an app to control the devices, but there were a few things that annoyed him. We talked about it, and half-jokingly, he asked if I could build a better one.

That sparked my curiosity, and just for fun, I started hacking something together. I quickly realized how much I enjoyed it and got increasingly interested in the topic. Eventually, I even bought my own smart home devices so I could use the app myself. Over time, I kept adding features and maintaining it.

After spending a lot of time on the app—and using it almost daily, especially in winter—I thought it would be awesome if others could download and use it too. So I defined a target feature set and a list of bugs to fix before it would be ready for the Play Store. Once everything was in place, I polished and prepared the app for release. After pushing through the (annoying but mandatory) closed testing phase, the app was finally published on March 30, 2025 🔥.

---

## Technical Stuff

### Key Data

| Category         | Answer         | Comment |
|------------------|----------------|---------|
| Target Platform  | Android        | My friend and I both had Android devices |
| Technology       | Flutter/Dart   | I had used Flutter in university and found it great for building apps that don’t need native features |
| RaspberryMatic API | XML-API     | I chose it because it was simple and had all the functionality I needed: https://www.homematic-inside.de/software/xml-api |
| Design           | Figma          | A powerful and free tool, great for creating icons and mockups |

The first few commits were pretty hacky—just me playing around with the API. But I quickly realized that if I wanted to continue working on the app, I needed a better architecture to keep the code maintainable.

---

### Supporting New Devices

One of the early challenges was how to efficiently support new device types. Using the XML-API, each device has specific attributes. For example, a thermostat might have `ACTUAL_TEMPERATURE` and `SET_POINT_TEMPERATURE`. These vary depending on the device, so hardcoding them wasn’t an option.

The data for a device for example looks like this:
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

My solution was to introduce a **mapping file** I call a **Driver**. Each device type usually supports the same features (e.g., read/set temperature), so I created a mapping of raw attributes to standardized custom properties used throughout the codebase.

When adding a new device:
- If it’s a supported type, I just add a mapping file.
- If it’s a new type, I also implement the UI and functionality.

---

### State Management

Initially, I used `setState` and `ChangeNotifier` to manage state. It worked but quickly became messy and hard to test.

Then I discovered [`flutter_bloc`](https://pub.dev/packages/flutter_bloc), a state management library that looked promising. I refactored the app to use bloc, and the code became much cleaner and easier to test.

---

### Security

One of my priorities before release was handling credentials securely. Since the app connects to the CCU, the only real security concern was storing login data.

To solve this, I used [`flutter_secure_storage`](https://pub.dev/packages/flutter_secure_storage), which stores AES-encrypted key-value pairs using each platform’s secure storage (e.g., Keychain on iOS and Keystore on Android).

---

## Release Process

The release process was a bit annoying. Since I'm doing this as a hobby and don’t have a company, I used a personal developer account.

To release the app, I had to:
- Find at least **12 testers** who would **opt in for 14 consecutive days**. It wasn’t clear what "opt-in" meant, but apparently it just means having the app installed. That's it.
- Create a **privacy policy website** for the app. I used **GitHub Pages** for this, which I highly recommend—it's fast, easy, and free.
- Fill out the usual stuff in the Play Console: app descriptions, screenshots, privacy settings, content ratings, etc.

After some back and forth, and a few reviews by Google, everything was approved, and I could finally release it to production 🥳.

---

