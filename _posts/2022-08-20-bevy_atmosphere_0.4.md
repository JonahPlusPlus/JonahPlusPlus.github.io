---
layout: post
title:  "bevy_atmosphere 0.4"
date:   2022-08-20 14:13:41 -0400
excerpt_seperator: <!--more-->
---
{% include image.html image="/assets/images/2022-08-20/bevy_atmosphere_splitscreen.png" caption="Figure 1" %}

Hey, Jonah here. With the release of bevy_atmosphere 0.4 (for bevy 0.8), I thought I would take the time to write a short article about the experience and technical details. Source code can be found at [JonahPlusPlus/bevy_atmosphere]("https://github.com/JonahPlusPlus/bevy_atmosphere").
<!--more-->

bevy_atmosphere is a procedural sky plugin for the Bevy game engine. It offers complete customization of the sky simulation, allowing for any number of effects (see Figure 1).

The focus for this update was to take the opportunity to revamp the internals, to make it more flexible and simple to use, while also improving the performance. Previous versions of bevy_atmosphere were limited, for instance, offered no support for split-screen apps. The sky shader also ran every frame, which was often redundant when the material was not changing, and had very noticeable color banding.

{% include image.html image="/assets/images/2022-08-20/bevy_atmosphere_shader_modules.png" caption="Figure 2" %}
