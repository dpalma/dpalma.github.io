---
layout: post
title: Driving straight without a DriveTrain block with ModKit for VEX
category: VEX
comments: yes
---

This project uses the separate motor blocks instead of a DriveTrain to move the bot.
Using separate motors allows greater control over the bot&#39;s movement. For example,
the bot can move smoothly while reading a sensor, like the bump or distance sensor,
to determine when to stop.
<!--more-->

The ModKit DriveTrain block is convenient and easy to use, but it can produce
jerky movement when trying to read a sensor to determine when to stop. Using
two separate Motor blocks instead of a DriveTrain will fix this, but there are
trade-offs.

First, Motor blocks require input as degrees or revolutions. To complete a
maze challenge, it is far better for a program to take inches or centimeters as
input. We will address that in a future session.

Second, it is easy for the motors to get out of sync. This can cause the robot
to veer to one side instead of moving straight. This [support link]( http://www.vexiqforum.com/forum/main-category/official-answers-ask-the-vex-staff/8035-motors-get-out-of-sync-intermittently-modkit-for-vex) describes the problem and gives a solution. Each motor&#39;s
&quot;move forward&quot; event listener increments a variable after the motor block completes. The brain
waits for this variable to get to 2, meaning that both motors are done moving.
Keeping the motors in sync this way helps keep the bot moving straight.

[Printable instructions here](https://docs.google.com/document/d/1xyqCGnXFV108BDxCAwV-JII8apqHsoCg7HV4J7YwCPI/pub)
