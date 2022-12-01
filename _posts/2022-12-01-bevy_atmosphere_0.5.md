---
title:  'bevy_atmosphere 0.5'
layout: post
date:   2022-12-01 12:20:30 -0400
excerpt_seperator: <!--more-->
image:  '/assets/images/common/bevy_atmosphere_logo.svg'
---

Hey, Jonah here. I'm releasing [bevy_atmosphere 0.5](https://crates.io/crates/bevy_atmosphere) for bevy 0.9, so I made this blog post to go over some of the changes.
<!--more-->

{% include image.html image='/assets/images/2022-12-01/gradient-example.png' caption='Figure 1: bevy_atmosphere 0.5 "gradient" example' %}

## Model-Pipeline Decoupling

The focus of this update is to set the groundwork for making more atmospheric models in future updates.
The Nishita model built into the plugin is fine, but I would like to support more accurate models, as well as stylized models.

So, the largest change of this update is the decoupling of model data from the pipeline.
I achieved this by removing the `Atmosphere` resource, which stored parameters for the Nishita model, and replaced it with the `AtmosphereModel` resource, which holds a `Box<dyn Atmospheric>`.
The definition of `Atmospheric` looks like:
```rust
pub trait Atmospheric: Send + Sync + Reflect + Any + 'static {
    fn as_bind_group(
        &self,
        layout: &BindGroupLayout,
        render_device: &RenderDevice,
        images: &RenderAssets<Image>,
        fallback_image: &FallbackImage,
    ) -> BindGroup;

    fn clone_dynamic(&self) -> Box<dyn Atmospheric>;

    fn as_reflect(&self) -> &dyn Reflect;

    fn as_reflect_mut(&mut self) -> &mut dyn Reflect;
}
```

The `clone_dynamic` and `as_reflect*` methods allow me to work with it as a trait object and get type data from the type registry.

The `as_bind_group` method is a bit of a hack.
Bevy's `AsBindGroup` trait can't be made into a trait object because of the `bind_group_layout` associated function.
So, I split it into two trait, `Atmospheric` and `RegisterAtmosphereModel`.
I plan on making a PR to split the `AsBindGroup` trait into two (or three, you have noticed the signature of `as_bind_group` isn't the same as the original; that's due to constraints with trait objects), so I don't have to implement their behavior myself.
The definition of `RegisterAtmosphereModel` looks like:
```rust
pub trait RegisterAtmosphereModel: GetTypeRegistration {
    fn register(app: &mut App);

    fn bind_group_layout(render_device: &RenderDevice) -> BindGroupLayout;
}
```

The `register` function allows for registering the model from `App` through the `AddAtmosphereModel` trait like:
```rust
app.add_atmosphere_model::<YourModel>();
```

Both `Atmospheric` and `RegisterAtmosphereModel` can be derived through the `Atmospheric` derive macro.
It has attributes similar to `AsBindGroup`, but also provides the `external` and `internal` attributes for specifying the shader, as an external or internal asset, respectively.

Now that we have these traits, how do they tie into the pipeline?

Well, when you register the model, `RegisterAtmosphereModel` stores type data of `AtmosphereModelMetadata` inside the type registry.
`AtmosphereModelMetadata` stores the type ID, the bind group layout and the ID of the cached compute pipeline (the loaded shader).
The pipeline can then use the `Atmospheric` trait object to get this type data and use it to render the model.

Looking up type data can be a bit slow, so instead of looking it up every frame, I use a hidden `CachedAtmosphereModelMetadata` to store the results, and whenever the model updates, I can check if the type ID has changed, and if so, then the type registry is searched for the respective type data.

## Gradient Model

With the basic parts of this system in place, I decided to test it out by creating two models, `Nishita` and `Gradient`.
Nishita was the model used by the old `Atmosphere` resource.
`Gradient` is a new model, that provides a simple gradient of three colors, the ground, horizon, and sky (see Figure 1).
In a future update, I want to make it more powerful by supporting "unlimited" colors and allow users to choose the method of interpolation.
However, Bevy's `AsBindGroup` doesn'tt support storage buffers, so I just have to wait till then.

{% include image.html image='/assets/images/2022-12-01/gradient-formula.png' caption='Figure 2: formula for a linear gradient of three colors' %}

Anyways, the model just mixes between colors based on a simple formula designed for three colors (see Figure 2).
It's fast and simple, but it doesn't give a ton of artistic control.
Still, it works for simple stylized scenes, and serves as a good starting point for any programmer who wants to create a more advanced stylized model designed for their particular artstyle.

With this new model, I was able to confirm that the new design works, and you can change out models at runtime.

## System Params

While the `AtmosphereModel` resource is useful for systems that match different models, it isn't the most idiomatic way of getting a particular model.

That's where the `Atmosphere<T>` and `AtmosphereMut<T>` system params come in.
Instead of casting the model yourself, you can specify the model you want to work with, like `Atmosphere<Nishita>` and it will cast it for you.

Later, I want to make it possible to write `Option<Atmosphere*<T>`, but that requires Bevy to provide some sort of trait that I can implement for `Atmosphere*<T>`, since I can't implement `SystemParam` for `Option<Atmosphere*<T>>`, due to orphan rules.

## Examples

I've updated and added some examples, including some new examples showcasing off the new models.
I've also updated the old examples to use my new [bevy_spectator](https://github.com/JonahPlusPlus/bevy_spectator) crate, which is a powerful alternative to bevy_flycam, since it allows for multiple spectators and windows, as well as alternative movement speeds and not being framerate-dependent.
With it, the splitscreen example now supports movement for both cameras, which you can switch between using the 'E' key (there is a small edge case I didn't notice, where you have to press 'E' and click the screen before you can move).

Anyways, that's pretty much it for this update.
A big thanks to the Bevy discord, which is always so friendly and helpful.

You can find the source code on [GitHub](https://github.com/JonahPlusPlus/bevy_atmosphere).

{% include heading.html heading="0.5 Change Log" %}

- Removed the `Atmosphere` resource in favor of the `Nishita` model.
- Added the `AtmosphereModel` resource, which holds an `Atmospheric` model.
- Added the `Atmospheric` trait and derive macro, which is used to define a model for the pipeline to render.
- Added the `Nishita` model, which provides Rayleigh and Mie scattering.
- Added the `Gradient` model, which provides a simple linear gradient of three colors.
- Added the `Atmosphere` and `AtmosphereMut` system params, for working with a specific model.
- Added `AtmosphereSettings.dithering`, which allows for enabling/disabling dithering at runtime.
- Updated `bevy_atmosphere::prelude` to include new common types.
- Added `AtmosphereModelMetadata`, which is used to store type data about a model.
- Added `AddAtmosphereModel`, which is used to easily register new models from an `App`.
- Added `RegisterAtmosphereModel`, which is used to register the model it's implemented for.
- Added `AtmosphereImageBindGroupLayout`, which is used to store a common bind group layout for all models.
- Added `SkyBoxMaterialKey`, which is used to pass the dithering state to the pipeline.
