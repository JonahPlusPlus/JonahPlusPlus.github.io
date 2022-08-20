---
title:  'bevy_atmosphere 0.4'
layout: post
date:   2022-08-20 14:13:41 -0400
excerpt_seperator: <!--more-->
image:  '/assets/images/2022-08-20/bevy_atmosphere_splitscreen.png'
caption: 'Figure 1: bevy_atmosphere 0.4 "splitscreen" example'
---

Hey, Jonah here. With the release of [bevy_atmosphere 0.4](https://crates.io/crates/bevy_atmosphere) (for bevy 0.8), I thought I would take the time to write a short article about the experience and technical details. Source code can be found at [JonahPlusPlus/bevy_atmosphere]("https://github.com/JonahPlusPlus/bevy_atmosphere").
<!--more-->

bevy_atmosphere is a procedural sky plugin for the Bevy game engine. It offers complete customization of the sky simulation, allowing for any number of effects (see Figure 1).

The focus for this update was to take the opportunity to revamp the internals, to make it more flexible and simple to use, while also improving the performance. Previous versions of bevy_atmosphere were limited, for instance, offered no support for split-screen apps. The sky shader also ran every frame, which was often redundant when the material was not changing, and had very noticeable color banding.

{% include image.html image='/assets/images/2022-08-20/bevy_atmosphere_shader_modules.png' caption='Figure 2: new shader modules for atmosphere types and math' %}

There was also a disconnect in the naming convention ("Mat" suffix instead of "Material") and shader language choice (GLSL instead of WGSL) from the rest of the community.
The first thing I did was rewrite the shader in WGSL.
Besides being more Bevy-like, it also gave the benefit of shader modularization (see Figure 2).
This allowed me to organize parts of my shader (data types and logic) and allow for crate users to create their own sky shaders by importing my shader modules.

{% include image.html image='/assets/images/2022-08-20/equirectangular.png' caption='Figure 3: one of my equirectangular projection experiments' %}

For optimization, I wanted to make sure the shader is only run when the material is updated, instead of rendering every frame.
So instead of rendering to the screen, the material would be rendered to a texture, that would then be used in a faster material.
The tradeoff is that instead of just rendering the pixels on the screen, you have to render pixels off the screen as well.
This trade off is worth it, since you wouldn't expect the material to be updated every frame, but can be updated occasionally, with no perceivable difference.
Once I get the texture though, I need something to display it on. I was using a sphere to render the sky before, so I considered using equirectangular projection (See Figure 3) to map the texture back to the sphere, but that was convoluted.
Instead, I decided to move to a skybox, and create a 6:1 cubemap for the 6 faces.
