---
layout: post
title:  "The Primordial Ooze: Part 2"
date:   2026-04-13 20:18:23 -0400
categories: otomo
tags: [otomo, embedded, rust]
---

Previously:

[Part 1](/_posts/2026-04-13-primordial-ooze-part-1.md): Introduction and explanation

# Otomo v1 (2020 - 2021) - Teleop via bluetooth, protobuf and embedded Rust

Just before the pandemic, I was working on a project at work that involved transmitting/receiving protobuf-encoded messages between an embedded system and an Android app. I had extremely minimal Android development experience, so I used the opportunity to create a simple Android app to drive around the little toy rover that I had purchased a year before. 

The other thing I really wanted to get into at the time was Rust. I'd been bound to legacy firmware development in C for several years at that point, and I desperately wanted something new. Something exciting, that also fit into my embedded niche.

So the first version of "otomo" that actually moved was a teleop system on a toy chassis with some custom additions and a scarily bad Android app.

![otomo-001](/assets/images/primordial-ooze-part-2/otomo-back.jpg)

The core of the robot was the venerable STM32F4-Discovery board, married to an expansion board that I received from a professor years before. That F4-discovery/expansion board setup has been in every successive generation of otomo since.

![discoboard](/assets/images/primordial-ooze-part-2/discoboard.jpg)

The original DFRobot Baron chassis was clearly designed with an Arduino and motor shield in mind, with holes predrilled for mounting it. However, I really wanted to avoid using an Arduino so I could use the STM32F4-Discovery board. So I ended up designing some more "plates" to add layers to the thing. One for the batteries, one for the Discovery board, and a layer for a breadboard and motor controller.

![front-layers](/assets/images/primordial-ooze-part-2/otomo-front.jpg)

Thankfully, I have a friend who was working at a place with a laser cutter at the time, so the plates were largely made of scrap acryllic cut into the shapes for the plates

![top-plate](/assets/images/primordial-ooze-part-2/top_plate.png)

I also ended up creating a motor controller board based on the L293 H-bridge and soldered it down to a janky little breadboard.

![motor-controller](/assets/images/primordial-ooze-part-2/motor-controller.jpg)

## Programming

At the time, I was working pretty much entirely with embedded C, and I wanted to try out something new to keep my skills fresh. At the time, Rust was becoming the hottest thing around, and was just starting to be used on embedded devices.

I toyed around with several different crates and settled for RTIC. At the time, it was really the only framework that was ready for use within the embedded rust ecosystem. Embassy was still in the _earliest_ stages of development, and async traits had yet to be ported to corelib.

The funny thing is that this started an interesting feedback loop with my job. I used my embedded Rust experiences to learn the language, which then landed me a job with a new R&D team that was exclusively using Rust for new projects. The stuff I learned and was working with at the new team was directly applicable to Otomo, so it was fed back

To control the robot, I created a simple Android app ([OtomoAndroid](https://github.com/TWALL9/OtomoAndroid)) to transmit messages over bluetooth to a BLE/UART converter chip connected to the robot. I leaned on protobuf to create the messages that would be transmitted ([otomo_msgs](https://github.com/TWALL9/otomo-protobuf)). The app created a simple joystick that fed commands in two parts

- throttle: the distance from the joystick from the origin
- orientation: the angle away from "North"

The firmware would receive these and after deserialization, have to translate the commands to left/right tank controls.

https://github.com/TWALL9/otomo-rs/blob/5c3370ef28ac891782b8827f8e7628dca40bdb63/src/motors.rs#L25

```rust
use defmt::Format;

#[derive(Debug, Clone, Copy, Format, Default, PartialEq)]
#[allow(dead_code)]
pub enum MotorDirection {
    Forward(f32),
    Backward(f32),
    Brake,
    #[default]
    Release,
}

pub trait OpenLoopDrive {
    fn drive(&mut self, direction: MotorDirection);
    fn current_direction(&self) -> MotorDirection;
}

/// Returns left/right motor inputs based on a unit-circle joystick.
/// [based on this example's drive output graph]: https://robotics.stackexchange.com/questions/2011/how-to-calculate-the-right-and-left-speed-for-a-tank-like-rover
/// Uses a very basic linear equation solver to plot drive power
/// The equations are calibrated on a 50% throttle input, so when turning the speed can be more
/// than 100%.  This function saturates the output speed to prevent that.
/// * `speed` - Percentage of radians in a unit circle describing the joystick
/// * `deg_heading` - Heading in degrees, with 0 as north.
pub fn joystick_tank_controls(speed: f32, deg_heading: f32) -> (MotorDirection, MotorDirection) {
    if speed <= 0.0 {
        return (MotorDirection::Release, MotorDirection::Release);
    }

    let r = speed.clamp(0.0, 100.0) / 100.0;
    let theta = -1.0 * (((deg_heading + 180.0) % 360.0) - 180.0);

    let left_raw = throttle_solver_l(r, theta);
    let right_raw = throttle_solver_r(r, theta);

    let left = if left_raw.is_sign_positive() {
        MotorDirection::Forward(left_raw.clamp(0.0, 1.0))
    } else {
        let abs_l = left_raw * -1.0;
        MotorDirection::Backward(abs_l.clamp(0.0, 1.0))
    };

    let right = if right_raw.is_sign_positive() {
        MotorDirection::Forward(right_raw.clamp(0.0, 1.0))
    } else {
        let abs_r = right_raw * -1.0;
        MotorDirection::Backward(abs_r.clamp(0.0, 1.0))
    };

    (left, right)
}

fn throttle_solver_l(throttle: f32, theta: f32) -> f32 {
    let (x_pos, y_intercept) = match theta {
        t if (-135.0..=45.0).contains(&t) => (t, throttle),
        t if (-135.0..-180.0).contains(&t) => (180.0 + t, throttle * -1.0),
        t if (45.0..=180.0).contains(&t) => (-180.0 + t, throttle * -1.0),
        _ => return 0.0,
    };

    let slope = y_intercept / 45.0;

    slope * x_pos + y_intercept
}

fn throttle_solver_r(throttle: f32, theta: f32) -> f32 {
    let (x_pos, y_intercept) = match theta {
        t if (-45.0..=135.0).contains(&t) => (t, throttle),
        t if (135.0..=180.0).contains(&t) => (-180.0 + t, throttle * -1.0),
        t if (-180.0..-45.0).contains(&t) => (180.0 + t, throttle * -1.0),
        _ => return 0.0,
    };

    let slope = -y_intercept / 45.0;

    slope * x_pos + y_intercept
}

```

It's based on a [post that I found on StackExchange](https://robotics.stackexchange.com/questions/2011/how-to-calculate-the-right-and-left-speed-for-a-tank-like-rover) where the theory is that a tank-control robot should balance its total thrust on the intersection of the left and right motor, adjusting the slope of control rather than driving them with no relation to each other

Naive control:
![naive control](/assets/images/primordial-ooze-part-2/naive-control.png)

Proper control
![proper control](/assets/images/primordial-ooze-part-2/tank-control.gif)


## Hardware Woes

Let's talk about encoders.

Glossing over how [typical quadrature encoders work](https://en.wikipedia.org/wiki/Incremental_encoder#Quadrature_outputs), and just about every quadrature encoder on the market works...they all have two channels for indicating speed, and direction.

_Except_ the encoders on this stupid robot. I really wish I could find pictures of them but I've since given this chassis away. There were two major flaws:

1. The encoders were mounted on the output shaft of the gearbox, rather than the rotor. This reduced the counts per revolution from somewhere in the hundreds to...12.
2. The encoders only had one channel. So I could determine speed, but not direction.

They looked like this, but instead of Hall effect sensors and a magnet, it was an optical break-beam and a janky PCB that looked like a cog screwed into the gear shaft.

![jank motor](/assets/images/primordial-ooze-part-2/crap-motor.png)

There were also only two encoders for the four motors. This _really_ reduced the ability to control the motors autonomously, without being able to reliably determine the direction or speed of half the motors, and not at all for the other two. 

I remember this really upsetting me, because I wanted the robot to explore spaces autonomously and to do very fancy motor control with it. My project notebook at the time reflects this pretty well lol:

![notebook](/assets/images/primordial-ooze-part-2/angry-notebook.jpg)

To think that I was also about to change all the odometry to an IMU of all things...

This is where I learned an important lesson that's going to appear a _bunch_ throughout all the writing I do going forward:

***A robot is only as valuable as its hardware integration***

## Moving On to Greener Pastures

Sure, you _can_ run SLAM entirely in firmware. Roombas use crappier microcontrollers than I was using, and can navigate entirely by dead reckoning. But, if you crack open a Roomba, you will find that they have adequate encoders. You need at least one "good" sensor for navigation. Either your wheel odometry is good, or your object-detection and ranging system is good. I had neither of these things.

I was trying to determine what I could do about this situation to improve the robot: 

- Adding a Raspberry Pi as a head unit would require more power control than the simple battery that I had
- Changing micro to an ESP32 for native bluetooth comms would require moving away from embedded Rust (there was no viable peripheral crate for Espressif boards at the time, and I didn't feel like writing it just to make a hobby robot work)
- The motors needed to be replaced, which would also change the body quite extensively.

In the summer of 2022, I started really looking into ROS for doing most of the heavy lifting for SLAM. I was really getting into this robot. And I noticed an opening at a robotics company that heavily contributes to ROS for an embedded developer. 

I'm not saying that I took the job _purely_ so that I could continue working on my robot with better knowledge, but I'm also _not_ not saying it.
