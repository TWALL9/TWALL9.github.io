---
layout: post
title:  "The Primordial Ooze: Part 3"
date:   2026-04-18 16:00:00 -0400
categories: otomo
tags: [otomo, embedded, rust]
---

Previously: 

[Part 1](/_posts/2026-04-13-primordial-ooze-part-1.md): Introduction and explanation

[Part 2](/_posts/2026-04-14-primordial-ooze-part-2.md): Making a robot sandwich named after samurai

# Otomo V2 (2022 - 2023): ROS 1, Teleop over the Web

# This is the story of how I got married

In November 2022, I started working at OTTO Motors. Their sister company, Clearpath Robotics, was a big name in the ROS community, and as such, OTTO used a lot of ROS stuff. So I started learning ROS on the side to keep up with what was going on at my new job. 

Around this time, my (then) fiancee and I were planning our wedding (for the second time. Covid really did a number on our nuptuals). Since we were having a no-kids-allowed wedding, we were trying to figure out what to do about the flower girl and/or ring-bearer situation. We figured that one of our friends or siblings could skip merrily down the aisle flinging paper flower petals for all, but then the thought occurred to me. 

_What if I used the robot?_

The plan took shape:

- The robot would have to be pretty large in order for the audience to see it, but small enough to be cute
- The robot would need to be tele-operated. We were _not_ risking the thing driving autonomously down the aisle and veering off course (more on that later lol)
- The robot would have some mechanism to fling confetti or flower petals or _something_ into the audience

## Mechanical Design

Based on the "cute, and medium size" requirement, I started sketching plans on what this robot would look like. One of the things I noticed while working at OTTO is that while all the robots were generally big metal rectangles, the smallest moddel, the OTTO 100, is cuter than the others. It has a very simple anthropormorphism in the fact that its center of rotation is near the back.

![o100](/assets/images/primordial-ooze-part-3/o100.png)

This gives it the appearance of having rear legs. Or at least pivoting like a little creature. While it's not ideal for a differential drive robot to have a center of rotation anywhere aside from the center of geometry (it seriously increases the navigation footprint when turning in place), it does make the thing look more alive than the typical roomba.

![drawings](/assets/images/primordial-ooze-part-3/drawings.jpg)

You can see that adding a little smiley face and googly eyes was a necessary requirement from day 1. Googly eyes add 10,000 cuteness points.

Next came what to make the thing out of. If you were able to read my chickenscratch notes you'll see what the most readily available option was: plywood. This robot was going to be large. Too large for a 3D printer (or any 3D printer that I had access to) and far too large to try and bum acryllic cutting of the same friend who did the work for the earlier Otomo. The choice of using plywood was also driven by the tools I had available to me: a drill, and a circular saw. The material is cheap enough to make mistakes on, and flexible enough to make damn near anything. It's of course _readily_ available here in Canada. Honestly I'm not sure why we make hobby robots out of anything else. If you have the tools I highly recommend doing this. The other option I was thinking about was to take a plastic storage tote and drill holes in it, but the caveat here was that drilling holes in thin plastic is always an awful messy process and the totes that I could find were:

- Just not the "right" dimensions
- Had non-uniform bases for mounting the internals
- Were too flexible

So plywood. Considering it didn't need to really carry any weight, I opted for simple 1/4" veneer board, with 2x2 to hold the corners up. It's cheap, and awful to cut across the grain, but it really just needed to be "a shape" rather than eye-pleasing, because the plan was to put a felt cover over it that we could decorate.

For the dimensions, I wanted to keep it to a single sheet of 24"x48" plywood, and to have dimensions that appear "natural". This resulted in the thing being 55cm long, and 33cm wide. This was largely because I found that the "cuteness" factor peaked if it followed the golden ratio. I have no idea where I came up with that idea and I'm not going to go hunting for it.

---

An aside: the measurements from here on out are going to make _even less_ sense. Because Canada. Due to our proximity to the largest (and really only) exporter of the imperial system, and our propensity to export lumber to them, we measure all our carpentry and woodworking in imperial. For most things _it makes sense_. It's pretty easy to measure a prime number/2^something. Everything is fractional. Except when you're bolting circuit boards to wood. Everything other than lumber is metric. So I'm trying to drill holes for M2.5 and M3 bolts using mm offsets on a board that arrived in imperial, which I'm cutting in metric. It's a disaster.

---

The other reason I put the drive wheels at the back was to reduce the number of points of contact with the ground. It's counterintuitive, I know. But my reasoning for this was simply that the wedding was at an outdoor venue, and the less hard plastic casters rattling on cobblestone, the better. Also because with the tools available to me, and the very real possibility that the drive wheels, motor mounts and casters were not going to be at the same height without shimming, having an additional pair of casters would cause the body to rock back and forth, which would have been really annoying to work with.

![front](/assets/images/primordial-ooze-part-3/front-panel.jpg)

The idea was to place the important components on the bottom, and just wire them however I wanted with enough space. This made things massively easier to work with compared to the sandwich bot, because I didn't have to worry about wires going through channels or whether wires had to go up and then down, everything was just spaghetti, which would be perfectly acceptable since nobody but me would see the inside.

![spaghetti](/assets/images/primordial-ooze-part-3/spaghetti.jpg)

At the top of the robot was an opening that I mounted a fan to. Specifcally, a fan that's supposed to cool one of these things:

![lifter](/assets/images/primordial-ooze-part-3/lifter.png)

It's a 24V fan, and certainly powerful enough to blast a quarter pound of confetti into a crowd. I did intentionally run this fan at 12V, to avoid making too much of a mess. In the end we didn't get to use it because the venue staff took umbrage with the idea of automated dispersal of papery well wishes. So the robot went from being a flower girl to a ring bearer.

As the box came together, and I was doing testing and drive tuning, I just kinda started calling it by a name: Dingus. This was mostly because I always thought "Otomo" was a little weebish of a name, and it's clearly not an automatic mower, or an ancient samurai clan. It's an idiot box that I'm using to drive stuff through my wedding. The name stuck, and while the repos are still named `otomo`, they're really for Dingus.

## Electronics

Dingus ended up pretty much as an expansion on Otomo:

* The Discovery board makes a comeback 
* Main computer added (Raspberry Pi 3)
* A rinky-dink home wifi router
* USB hub

These were all parts that I had lying around, so there wasn't really any drive to go through a selection process.

For the drivetrain I opted for a pair of 12V gear motors from Pololu with integrated encoders, and a pair of Toshiba motor controllers.

![controller](/assets/images/primordial-ooze-part-3/motor-controller.jpg)

![motor](/assets/images/primordial-ooze-part-3/motor.jpg)

I decided to spec the motor for torque, rather than speed. I used the load curve from Pololu to determine that this motor _should_ be operating near the peak of its efficiency with Dingus' estimated weight, based on the torque it could deliver to the roller blade wheels that I was using

![wheel](/assets/images/primordial-ooze-part-3/wheel.jpeg)

![torque](/assets/images/primordial-ooze-part-3/wheel-torque.png)

This did end up being _way_ on the side of torque rather than speed. Dingus was not a fast mover down the aisle, but it could bully my roomba with extreme predjudice.

(figure out how to put a video here)

## Power and Electrical

I ended up using no less than 3 different voltages throughout the system:

1. 12V for the motors and USB hub
2. 9V for the router
3. 5V for the raspberry pi, embedded system, and everything else.

This was done with a pair of lipo packs (3S and 2S, attached to a boost converter), and a USB power supply. The 3S battery really limited my runtime, but Dingus didn't need to run terribly long. Still though, it was a terrible system largely solved with parts that I had on-hand, rather than solving it the right way.

The actual power to Dingus was just done via hand-soldered XT-60 plugs, with buttons on the rear panel to power individual components. I modified the device tree of the rpi to include a power button tag, which was wired to one of the GPIO as an input.

I also added an e-stop button that I got from the scrap bin at work. Not for any actual functional safety reasons, it was merely acting as a cutoff switch for the motors, should things get out of hand. The button itself is definitely not designed to act as a real cutoff, but the power was low enough that I wasn't worried about any internal welding. Adding the button was definitely the most prescient thing I did when building Dingus, for reasons I'll get into later (subtle foreshadowing).

![rear-panel](/assets/images/primordial-ooze-part-3/rear-panel.jpg)

The router was the only device powered by the smaller 2S battery. I used a boost converter to bump the 7.5V from the 2S battery up to 9V so the router wouldn't pull too heavily. This was pretty much nonessential, but I didn't want the router to brown out, largely because it was pivotal to how I was planning on controlling the thing.

## Networking

And you may be right to ask yourself: "what is this doofus doing with a home router in a robot?" And I may be right to respond: "I didn't have better hardware for teleop"

The router was serving a local wifi AP, that the operator could connect to via their phone. IMO, the genius part was because the router has a much larger range than a phone hotspot, it would be easier for a phone to connect to the robot, rather than the router connecting to the phone.

That and it gave Dingus cute little bunny ears...coming out of its butt.

Internally, the router was connected to the raspberry pi over ethernet, and the raspberry pi connected to the MCU and camera over USB.

## Programming

This should probably be its own blog post, but I like the idea of these posts being a trilogy rather than quadrilogy. Name a good quadrilogy. They don't exist. Good things come in threes.

I did the original programming for this using ROS 1. Yeah, come at me, haters. Really I did this because it was what I was using at work at the time, and because I didn't want to learn two different types of ROS at the same time. And honestly, ROS 1 has an easier learning curve than ROS 2.

The software had several nodes.

[otomo_serial](https://github.com/TWALL9/otomo-serial): Strictly for communicating with the MCU over USB, it did the protobuf/ROS translation and creating joystick commands in the same manner as the Android app. The protobuf-encoded message stream was wrapped in KISS-TNC for message delimiting. Here's the crate I wrote for the MCU: [kiss-encoding](https://github.com/TWALL9/kiss-encoding)

[otomo_camera](https://github.com/TWALL9/otomo-camera): a dumpy OpenCV script that was largely created from a tutorial that publishes USB webcam images on a random topic.

[otomo_webserver](github.com/TWALL9/otomo_webserver/tree/d38022eb7a6f1bb7a7cb8c0ce29ad0160c0bc769): This package served a webpage using flask. I used `rosbridge-js` to bridge the topics from the internal network to the user for teleop. I used `nipple-js` for the joystick, which translated the x/y coords to a `Joystick` message (largely the same as the Android app). It also has my very most favourite piece of terrible code I've ever written:

```javascript
        var imageTopic = new ROSLIB.Topic({
            ros: ros,
            name: '/camera/image/compressed',
            messageType: 'sensor_msgs/CompressedImage'
        });

        imageTopic.subscribe(function(msg) {
            //console.log('got image');
            document.getElementById('image_pane').src = "data:image/jpeg;base64," + msg.data;
        });
```

```html
<body>
    <h1>Dingus View</h1>
    <!-- increasing size of image without inflicting compression lag on the Pi -->
    <div class="container">
        <div class="camera">
            <p><img id="image_pane" width="640" height="480"/></p>
        </div>
```

It literally updates the HTML with each MJPEG frame it receives over a topic. It's so satisfyingly awful.

The firmware architecture remained largely unchanged from the original system, using RTIC as a system framework and the stm32f4xx-pac crate. However, I did add PID control for the wheels, based on real actual encoder input from the MCU's QEI timer interface.

# Oh yeah, it's all coming together now

So far, everything is coming along swimmingly. The robot is acing the "real world" tests I was doing by driving it around the neighbourhood, over gravel, etc. Things are looking excellent.

And I've gotta say, it looked amazing in its official photoshoot

![closeup](/assets/images/primordial-ooze-part-3/closeup.jpg)

(my personal favourite part is the little flower we pinned to it)

Pre-ceremony, everything checked out. Batteries were full, networking was golden. One of my brothers was the designated operator and he passed his driving test with flying colours. Everything, EVERYTHING was set to go off without a hitch (except me, I was getting hitched).

# Oh god oh god, it's all falling apart now

Let me set the scene: you're standing at the altar, with your blushing bride to be. You are surrounded by your closest family and friends, for the wedding you've been waiting four years for, planned twice, and dealt with a global pandemic over. Nothing, NOTHING can ruin this moment.

"Please bring up the rings" says your best friend of nearly 20 years, performing the ceremony.

You hear a clattering, a rattling, as the little plastic casters turn into position. The felt box at the far end of the aisle, the product of months of work, comes to life. It toddles, a little slowly, down the aisle. It's a little loud, owing to the fact that the wooden body is hollow and rather rigid. But it's damn cute. Friends and family stand up to get a better view of the adorable little thing, probably the only robot they'll see at a wedding for a long time, approach its proud figurative parents.

Then it happens.

A caster hits a cobblestone weird. 

The robot veers off course.

My eyes shoot to my brother, his eyes wide in fright as he stares at his phone, trying to steer it back on course, before looking up at me. No words need to travel between us, but the look says everything:

"Shit, it's not responding."

My other brother launches into action. He places it back on course, the robot's wheels have been stuck in a "full speed forward" command, and it finishes the rest of its approach up to me and my future wife. 

And it doesn't stop moving.

Here I am with a profound choice, and a choice that I feel is somewhat relevant to people who let their dogs be part of the wedding ceremony.

"I could let this thing ram my leg, and basically start humping it"

"Or I could pull up the little skirt on this thing, and spank the e-stop on its ass."

I hit the e-stop. Everyone laughed. My contribution to the wedding was over. The other engineers in the crowd (there were several) must have all felt that same panic and shame of a failed demo.

But, it's one of the parts of the wedding that people bring up later. And I'm only sad about it in retrospect. Sure, I could have gone about it entirely differently. No ROS, just a basic RC car via cheap consumer-grade hardware. I could have done the entire thing over bluetooth and an xbox controller, no webpage needed. I could have implemented something that in retrospect, was plainly obvious: if no command is received from the PC within X ms, stop the motors and unwind the PID. None of that really matters though, we had a pretty good time with it, even if it was simultaneously one of the happiest and most embarrassing moments of my life.

![wedding](/assets/images/primordial-ooze-part-3/madeit.jpg)

# The Reception

The funny thing is, that glitch didn't appear again during the wedding or reception. Dingus tore it up on the dance floor, without a single burst of lag. I did manage to replicate the issue later, and while I don't quite believe the theory, I leave it up to the reader to think on it.

The only real difference between the trials and actual wedding run was the presence of a ton of peopl, in close proximity. I left the wifi interface on the raspberry pi on. I think it may have been picking up hotspots, or just enumerating SSID's that were further away or part of the venue's network. I don't have a single log or rosbag of this, because why would I implement that for a hobby robot?

_It's not like that's what I do for my day job or anything_. Drivetrain control, diagnostics and monitoring systems.

Ah well, we all live and learn. 

# Coming back from the Honeymoon

That level of internal embarrassment is basically my villain origin story. I could have retired the thing right then and there. A job (sorta) well done. But there were so many things that I wanted to improve.

* The triple battery situation was hilariously bad
* The body was the worst piece of basic carpentry ever done by humankind
* I really, really wanted to get the thing driving on its own
* ROS 2 was something I figured I'd have to learn anyway.

So instead of putting it on the shelf, I decided that I'd keep working on Dingus. Actually learn higher level robotics rather than the firmware and controls land I was familiar with.

And that brings us to today-ish. Or 2.5 years ago.

I'm going to be uploading the rest of the posts for "Son of Dingus" not as a monolith, but as a contained system-oriented view. I've been working on the ROS 2 iteration for years, off and on. And I think it's pretty neat. We'll see if these posts keep my motivation up for working on it.

Thank you for reading the primordial ooze series. Hopefully it's coherent, much of this is from hazy memory and half-legible notes. Writing about it from the future, I'm realizing that the best amount of work on it stemmed from when I was motivated by a deadline. The original sandwich bot, otomo, was more or less me learning embedded rust and just hacking around. But looking back on Dingus, even if it was super simple (and quite honstly below my capabilities, even then), I have fond memories of it. The anticipation of making something that you _want_ to share with a crowd, the justification of buying the parts and tools necessary to make it, and just making something with your own hands is something I'd largely forgotten at that point.

Making a hobby robot like this is not that difficult. The most difficult part is spending the money on tools and parts. That's what really separated Otomo from Dingus. Otomo was built from toy mechanical parts, and Dingus was actually made of real mechanical parts. This goes into a challenge that a coworker and myself have been joking on and off about for a long time
