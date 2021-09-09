---
layout: post
title: Capturing Tuya IoT Events
date: 2021-09-09 00:00:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: posts/capture-tuya-events-webhook.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Tuya, Macrodroid, Webhook]
---

Smart automation in the Tuya app is great for basic control of your IoT products. But if you want to take more control you can capture your IoT events into your own webhook.

This article is written for geeks and tinkerers. Solutions such as Home Assistant and Tuya's built in Smart Automation exist and will handle most solutions for home IoT.

## Prerequisites

* Android Device / Android Emulator
* Tuya App
* Macrodroid or Tasker (Android App)
* Webhook receiver
* IoT device (such as door sensor/PIR sensor)


## Tuya Setup

With your Tuya app go to "Smart" from the toolbar and click the add button to create a new smart event. Choose the device and the event, then select to send a notification/message. What ever is typed into the message will be sent to your webhook URL.

In the example, if movement is detected by the PIR sensor then a notification will be sent. You must close / minimize the Tuya app for the notification to be sent as a phone push notification - which is critical for the next stage where Macrodroid will capture the notification.

![Setting up in Tuya]({{site.baseurl}}/assets/img/posts/tuya-create-notification-webhook.gif)


## Macrodroid Setup

When the Tuya app is not active and the device event is triggered a push notification is sent to the device. Macrodroid can capture this notification and send it as a webhook to your specified URL that can handle the event.

This step takes a small amount of experience with Macrodroid.

### Trigger

Notification received, typically I use a "contains" to ensure it does not pick up other notifications.<br>
Ensure "Prevent multiple triggers" is ticked to prevent duplicate events.

### Actions

* Set a local variable as `[not_title]`
* If required do text manipulation
* Make a `HTTP GET` call to your hosted webhook receiver passing the notification text
* Clear the notification

![Macrodroid setup]({{site.baseurl}}/assets/img/posts/macrodroid-iot-notification.png)


## Limitations

<strong>Buttons</strong><br>
Some devices like IoT buttons cannot be added through "Smart" so it's not easy to add a notification.<br>
"Tap-to-run" that can be attached to a button press don't support sending messages/notifications though a bug that I don't fully understand in the Tuya app is a old smart action that sent a notification became a "Tap-to-run" enabled action so I was able to bind a IoT button to a notification. When I can find out how to reproduce this edge-case I will update this page.

<strong>Tuya app loaded</strong><br>
If the Tuya app is loaded the notifications binded on devices won't be phone push notifications so Macrodroid will not be able to capture them. To test the full workflow the Tuya app must be running in the background and not foreground.

<strong>BlueStacks can crash</strong><br>
If you're using an emulator like BlueStacks expect it to crash occassionally. For me it crashes every other day and I've not found a solution to this problem. I plan to move everything to a real Android device when I'm out of the Home IoT PoC stage. To detect if BlueStacks has crashed I have a ping health check that my server will send to Macrodroid and expect a reply - if there is no reply within 2 minutes my health check fails and I get notifications to my personal phone device.


## My Setup

The Tuya notification messages are made up of the following parts.

```iot-event:id=ff72eb7e-87d6-45f8-8025-798899b271be:key=sensor-door-open:value=true```

{% highlight bash %}
//To ensure Macrodroid captures the correct notifications
iot-event:

//The UUID of the device in my local database
id=[UUID]

//An alias of the event name, in the database I have events such as sensor-door-open, light-on
key=[EVENT_NAME]

//State of device
value=[BOOLEAN, STRING]
{% endhighlight %}

For tracking sensors I will add two Tuya smart events. One for movement [and if the PIR Sensor supports it] and another for no movement (noone). The notification `id` and `key` will be the same but the `value` will be true if it detects movement and false if no movement is detected. If your PIR sensor does not have events for both movement and "noone" then throw away that PIR sensor and get one that does!

![List of Tuya actions]({{site.baseurl}}/assets/img/posts/tuya-smart-list.gif)