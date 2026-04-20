---
layout: post
title:  "State of the Dingus"
date:   2026-04-19 21:00:00 -0400
categories: otomo
tags: [dingus, ros]
---

# "Son of Dingus" (2023 - 20XX)

After the wedding, I wanted to re-use Dingus for actually learning how to robot. Really, I wanted to fix things about it that really nagged at me:

- The wooden box (not actually pictured in [primordial ooze part 3](/_posts/2026-04-18-primordial-ooze-part-3.md)) was made in the shoddiest of manners
- I wanted to use ROS 2
- I wanted to actually learn SLAM and how to navigate
- I wanted to use less than 3 batteries

I knew that if I were to build a robot just for shiggles, I would get distracted and put it aside at some point. To that end, I needed some requirements:

- I wanted to get the thing to navigate autonomously throughout a house
- I want it to integrate with the rest of a smart home, so I can tell it to move room to room
- I wanted to get it to automatically dock, and charge when the battery is low.

# The wooden box

Despite looking like ass at the time, the wooden box strategy was actually pretty solid. I still only had the tools that were used to create the first box, but at least this time I'd put more care into it. At least this time I had access to a table saw rather than a circular saw, so the cuts were straight

![front view](/assets/images/primordial-ooze-part-3/front-panel.jpg)

![rear view](/assets/images/primordial-ooze-part-3/rear-panel.jpg)

The internal wiring remained spaghetti and I'm not apologizing for it

![spaghetti](/assets/images/primordial-ooze-part-3/spaghetti.jpg)

# Updating to ROS 2, and upgrading the hardware

At the time, I was using a raspberry pi 3 (2017). I wanted to move to an LTS version of ROS 2, but that meant that I'd have to update the version of ubuntu being used on the Pi. I opted for Ubuntu 20 so that I could use Foxy. However, Jazzy would be coming out the next year (2024), and taking over as the new LTS. I also found that while Canonical _says_ that Ubuntu Server 20.04 runs on a RPI 3, that isn't really the case. The experience was slow, buggy, and I was having hardware issues like you would not believe trying to run any sort of camera, even a basic USB camera. Moving to Ubuntu 24.04 would not solve the problem.

To alleviate this, I actually ended up pre-ordering a raspberry pi 5. Yes, I'm well aware that I _should_ have bought almost anything else. A Jetson would have better compute, a NUC would have more power, but really the reason I stayed with the Pi ecosystem was inertia, and because I didn't want my computer to be a power hog.

For navigation, I finally bought a lidar. I picked up a basic RPLIDAR A1 unit, which I planned to connect to the Pi over USB. Super basic, super simple.

# Power pains

Finding a suitable way to power the robot has been a journey all on its own. With the Pi 5, I realized that either I would need a dedicated USB-PD power bank that could supply the Pi 5's weird 5V/8A PD configuration, or I'd need to power it over the USB port, and power the USB peripherals separately. Powering the peripherals separately would not be a problem with an externally powered USB hub, but this is where I found my trusty 3S battery would fail me. See, the 3S is _nominally_ a 12V battery, but it will dip down to 11.1V when drained. And the USB supply powers off at 11.6V. And the Pi would brown out when trying to power the USB hub. No good. I had set up 3 ADC channels on the MCU to read the battery voltage to try to alert me if the battery was getting critically low, but I found that the readings just weren't accurate enough (really this is because I chose poor resistors for the voltage dividers leading into the ADC's, blowing out their accuracy).

I was looking into using other batteries, USB-PD power supplies that could also supply 12V@3A for the motors, when the simplest solution struck me.

Drill batteries.

It hit me when I was at a friend's house. He'd converted his daughter's Power Wheels buggy to run on a pair of Milwaukee batteries, and that thing _ripped_. It hit me that I could do the same with Dingus using the Ryobi batteries I had at home. They're 18V, and can put out a crapload of instantaneous power. They're not high capacity (I had a couple 4Ah units) because they're designed more towards short bursts of high power rather than continuous draw, and they're more expensive than just buying a 5S lipo battery pack, but there were some benefits:

1. I already had a bunch
2. I already owned a charger
3. They have an internal BMS

That last point is crucial. Lipo packs have no controls against discharging them beyond the point of recovery. If you don't have a good way to measure the voltage of your battery in a robot, you _will_ run the battery down to the point of damaging it. This is what was starting to happen to my 3S battery. Whenever I ran it to browning out the Pi, I would have to play the trickle charging game with my charger. Sometimes it would throw critical low-voltage faults and I'd have to cycle it in and out of the charger until it got just enough juice to bypass that fault and actually start charging. Ryobi ONE+ and Milwaukee M18 batteries have internal BMS boards to handle this kind of thing.

The history on it is even kind of neat, at least for Ryobi. They're not great tools, but they're ubiquitous. And they were early to the cordless game. At the time, their batteries were powered by nickel-cadmium (NiCAD) or NiMH chemistries. As lithium became more available, the same chargers were suppowed to power the same batteries. However, the charging cycles for these chemistries are different, so the BMS units themselves perform the charge balancing and end-of-charge indication to the charger.

Now that I had anywhere from 17 to 21 volts from the drill battery, powering the rest of the parts was trivial: just a pair of buck converters: one at 12V and the other at 5V. The 5V converter even had a USB output to power the Pi 5. No issues since then.

# Running ROS 2 on the Pi 5

Son of Dingus was originally developed with ROS 2 Foxy on my laptop according to the _EXCELLENT_ tutorials from [Articulated Robotics](https://www.youtube.com/@ArticulatedRobotics/videos). The actual repos I'll detail later. 

Naturally, there was no Ubuntu 20.04 image available for the Pi 5 on release in 2023. So I created a Docker image that pulled all the packages I was using.

[otomo_docker](https://github.com/TWALL9/otomo_docker/tree/6fa3dd15b658d4a2c7437cdc4d0b452d803512e1)

The image had to be run in god mode, to connect to the `/dev/ttyOtomo` symlink I'd created for the MCU board.

After ROS 2 Jazzy was released, I ported the Dockerfile to Jazzy. And once the Ubuntu 24.04 image was stabilized for the Pi 5, I just ported over to using that.

Honestly, I'm not a huge fan of running Ubuntu on the Pi hardware. However, running ROS on raspbian (or Raspberry Pi OS, or whatever it's called now) is itself very finicky. If it weren't for the fact that I'm doing lots of edits to the config files on the Pi container itself, I'd go completely over to the container method.

# The Software

Some things are old, some things are new:

### The old
- The STM32F4 Discovery board is still there, still running RTIC
- Comms between the PC and MCU are still protobuf, wrapped up in KISS-TNC

### The new
- Gone is the old `otomo_serial` node, instead replaced with [otomo_plugins](https://github.com/TWALL9/otomo_plugins) and [async_serial](https://github.com/TWALL9/async_serial). 
- "handheld teleop" is done through an Xbox controller rather than over the web

`async_serial` is a package that I wrote that uses `asio` from boost to create a nonblocking read/write interface to the USB-CDC endpoint on the MCU. This is important as the ros2_control plugin in `otomo_plugins` itself needs to be nonblocking. 

`otomo_plugins` creates a `SystemInterface` rather than a `JointInterface` because there's only the single communication point to the MCU. All traffic flows through it, rather than having a communication endpoint per wheel.

## Communication

This is probably also a good point to introduce how I'm using protobuf. Or at least how I've been using protobuf for the last...5 years?

[otomo-proto](https://github.com/TWALL9/otomo-protobuf/blob/6c9dac0fa8c0499e83f47309b7ed783561833663/otomo.proto)

```protobuf

message TopMsg {
    oneof msg {
        Config config = 1;
        Joystick joystick = 2;
        FanControl fan = 3;
        RobotState state = 4;
        DriveResponse drive_response = 5;
        Pid pid = 6;
        Imu imu = 7;
        Battery battery = 8;
        DiffDrive diff_drive = 10;
    }
}

```

When using an embedded device, especially using serial, you don't have access to the protobuf `any` [message type](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/Any.html), since you cannot lookup what that message is at runtime. In this case, the best way to work is by using a `oneof` within a top-level container message type. This has a couple benefits and one major drawback:

### Pros
- Your messages are still quite small on the wire, a `oneof` only adds a single byte to the payload
- packing/unpacking messages is still very fast, faster than `any`.

### Con
- The struct representing the message in code is much larger.

Especially when using `prost`. The `TopMsg` type is an enum, which is as large as its largest member. So `TopMsg` in the firmware is gigantic.

## Mapping and Navigation

I'm mostly using whatever defaults come with Nav2 at this point, tuned slightly for the shape of the box. The reason I didn't get too into Nav2 settings was because Dingus was having a hard time navigating my old apartment.

- It had several very narrow hallways, and I kept the pivot point near the rear of the robot...for...reasons??
- The only open space was dominated by a massive, thick, shag rug. So thick that Dingus simply could not overcome the power needed to turn in it.

It left me with the only navigable areas being a space about as large as a dinner table, and my kitchen. Neither were great.

However, throughout 2024 I did manage to get it to map and navigate the areas that it could actually fit in. 

![old-apartment](/assets/images/state-of-the-dingus/living_room.pgm)

Dingus also made an appearance at the inaugural Ontario ROS meetup at Clearpath Robotics HQ, where he was a bit of a hit

(gotta find the picture of this)

I'm the guy absolutely powerstanced in front of the plywood disaster.

# The State of the Dingus

I have a confession: Dingus hasn't moved in nearly a year. The last time it moved was to create a map of the house my wife and I bought in 2025. While we were moving, I took the opportunity to map the entire place without any furniture

(main floor)
![new-home](/assets/images/state-of-the-dingus/new_home.pgm)

(upstairs)
![upstairs](/assets/images/state-of-the-dingus/upstairs.pgm)

The main reason it hasn't moved is: motivation

This is the same reason that the original Otomo robot didn't move for years at a time. It's because I didn't have a deadline like I did with the wedding. That and because I keep finding little fiddly things to fix, when in truth they are small enough issues that I could ignore them in favour of achieving the goals I set out to achieve.

So that's the real reason that I've finally set up a blog about it. If people _see_ that I'm a lazy mess, then there's _accountability_.

To close out, here's a picture of the current spaghetti. Accountability Spaghetabiliti

![accountability](/assets/images/state-of-the-dingus/accountability.jpg)
