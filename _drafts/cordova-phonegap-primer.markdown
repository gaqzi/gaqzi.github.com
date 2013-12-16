---
layout: post
title: What is Cordova/PhoneGap and lessons learnt from making one app for four platforms
categories: cordova phonegap programming javascript mobile
---

# Introduction

This is going to be an introduction to Cordova/PhoneGap with examples
from an app we recently delivered for four different platforms;
Android, BlackBerry 10, iOS and Windows Phone 8.

I'll by the end of it leave my personal comments creating these kinds
of apps as well as what I think are some good practices to have.

All opinions are my own and should not reflect upon the people who
give me money to sit at my desk every month.

# The magazine app we built

For our client we built a magazine style application which rendered
large images of the pages in their magazine for the user to view on
their device. This was because there was a constraint on getting the
content of the articles with images in any other format than their
print PDFs.

We were a team of three people who took this on;

* Designer [Cholo De Villa] who did the  UI/UX
* Backend engineer [James Rivett-Carnac] who did the Django
  rest-framework based backend
* Tie-it-all-together-with-Cordova full stack developer [Björn Andersson]

The big deal for this app was making sure there wasn't an
excruciatingly large download to get the magazines available for the
devices, the hope is that the app will be used on the go to read the
magazine which would entail downloading using the 3G network.

So we prepared 30 different sizes/pixel densities that each page was
rendered in. That is all from thumbnails, iPhone 4S/5, Samsung Galaxy
S4, iPad retina displays and for all those that doesn't match;
original size.

There is a lot of assets created for each upload and it takes a while
for our background jobs to prepare each magazine for publishing.

## Frontend

The actual frontend itself isn't anything spectacular, it's just all
about trying to make it as fast as possible on all devices we're
trying to support. And to get as close as possible when it comes to
feature sets. We had to disable some niceties on WP8 just to get it to
work, but c'est la vie.

# Why Cordova and not X?

I've made a couple of cross platform apps that has gone out in
production. Before I made the first one I had a look through the
ecosystem, early 2012, and it was between PhoneGap, [Appcelerator Titanium]
and RhoMobile.  And that time I ended up using [RhoMobile].

At that point in time it had support for all the features I wanted and
I could write most of my logic in Ruby which is the language I had
just spent five years writing software in, a match made in heaven.

We work through and ship our first app and at about that time
RhoMobile gets bought up and it starts dropping things I wanted, and
fixes for features that broke starts taking really long to get
done. So when time came to make another app I had another look at the
ecosystem and I was back at comparing Cordova and Titanium.

This time I thought about it and went with PhoneGap because Adobe is
sponsoring it and it looked as if they were doing it pretty
seriously. They don't have any web platforms on their own, browser
etc, and they just want to make sure everyone is interoperating to
keep their own tooling business running (Dreamweaver etc). So I
decided to put my eggs in Adobe's basket and so far I'm very happy
with my choice.

# What is in a name? Cordova and PhoneGap

Cordova is a system for packaging up HTML5 applications to run as
mobile apps with access to hardware features on the devices it's
running on. There's a bunch of defined APIs that are all the same
between the supported platforms, although that doesn't mean all
platforms work the same or have the exact same APIs. Depending on just
how popular the platform is to use the differences from the two main
platforms, Androic and iOS, might be quite large.

Cordova is the core open source project that is part of the Apache
Software Foundation, gifted there by Adobe after they bought Nitobi
the original creators of PhoneGap. PhoneGap in turn is a distribution
of Cordova made by Adobe that has some extra features and integrates
with Adobe's PhoneGap Build service.

So when people say PhoneGap they generally mean Cordova which
implements most of the features found in PhoneGap. 

# So what is available to me when using Cordova to build my apps?

The most up to date listing is available on the [platform support]
page, but the big bullet points are:

* Accelerometer - Is the phone moving?
* Camera - Still and video, of course also audio
* Compass
* Contacts
* Filesystem - Create and manage folders and files on your own,
  depending on the platform you can access other apps files and
  folders as well
* Geolocation - Where am I? There is talk about adding in proper
  geofencing support and others, but the basic of accessing the GPS
  and the current location is in now
  
## Plugins

But this is just the core platform, there's also a whole load
of [plugins available][cordova-plugins] available, for example:

* Barcode scanner
* Push notifications
* Calendar

And many, many more. The downside with the plugins is that they might
not be available for all your platforms. For instance the
standard [push plugin] available on [PhoneGap Build] is only available for
Android and iOS. And if you're going to use a plugin that is not part of
the Cordova core do put in your due diligence before you promise anything,
and that even holds true for the Cordova core.

## HTML5, CSS3 and Javascript support

So we're building a webapp and exporting that as a native phone
application, so what do we have available? The general answer is
pretty much the same as the default native browser on the
platform. But there's caveats to that.

* On Android versions prior to 4.4 it's the browser, but from Android
  4.4 and forward it's Chrome/WebKit
* On iOS it's the same as mobile Safari, but you don't get the same
  javascript engine. No JIT compilation, but to feel this you need to
  be doing some really fancy stuff in Javascript. And either way
  Apples platform has the best feel anyway in my experience.
* Windows Phone 8 has some really bastardized version of IE mobile
  running; it reports support for things it doesn't support (SVG),
  does weird CSS animations, and of course Microsofts [pointer events].
  
  Although I must admit that I think the pointer events are probably a
  great idea, it's just way different from what is available on every
  other platform. So if you're successfully using a bunch of tools and
  happen on WP8 you might be in for a surprise in the amount of code
  you've to rewrite to support pointer events.
  
But this also means you've to figure out how far down you want to go
with support different devices/versions of the different platforms. If
you're going to support Android 2.2 you're going to run into issues
with a bunch of missing CSS features.

### Your grandfathers CSS support

For example I spent a day figuring out why images were blurry on
Android. Which turned out to be because of the, to me at the time
unknown, CSS directive [backface-visibility] which was supposed to be
set to hidden. And when I had that applied on devices that didn't have
CSS 3D transforms and a pixel density of more than 1 I couldn't get
iScrolls zoom functionality to work.

(backface-visibility is used in 3D transforms and if you flip the
element over it'll show a mirrored copy of the image on the back, as
if it had been rotated and you were looking at it from
behind. [A JSFiddle with an example.][backface-visibility-example])

So this is the kind of thing you're bound to have to deal with, but if
you've been doing mobile websites you probably know of some of them by
now and won't be spending an insubordinate amount of time on it. I'm
lucky to have a very good designer on my team that takes care of all
the normal cases, so I can spend my time dealing with the really weird
ones.

# But what about performance

Let's make it clear; you won't get the same performance as a well
polished native app. But you can make it damn close if you put in the
effort.

For instance on the iPhone you can do very well with the newer
releases excellent hardware accelerated CSS support. Newer Android
devices also do very well just because there's a silly amount of
hardware backing everything, but for older Android devices you'll have
to gracefully degrade as far as possible.

The people who use Android devices also matters for your target
audience, even if Android is outselling its competitors the question
is if it used as much. Or if it's just the cheapest phone possible
being used as the new calling and texting machine, just with a cute
robot all over the interface? I know that's the way my mother views
her Android.

And a lot of opinions on Cordova is from way back in the days, and two
years ago is really the bad old days by now. Comparing a Samsung
Galaxy SII with the SIII and then the S4 is crazy, because of the
sheer difference in what the devices can do. The only weird part is
how Apple manage to make a phone that is 2+ years old still be viable,
I personally use a 4S and it's doing very well for me.

# How do you use Cordova?

Cordova is from version 3 really easy to use, the entire frontend you
use is done by [NodeJS] and you got excellent [CLI] support. The
entire core has been moduleraized and core of Cordova is very small,
and all the big part API features you need are installed as
plugins. Which means that a critical bug affecting for instance the
filesystem API can be dealt with as a point release for that plugin
instead of having to release a complete release of Cordova.

This also means that the entire plugin infrastructure that you use to
write your own plugins with is used every day by all other developers
using Cordova. It's not something that'll just break all of a sudden.

So for example if you install Cordova on your computer and want to
create a new project and get it running for your emulator:

```
> npm install -g cordova
> cordova create example-folder org.example.my.app.id MyAppName
> cd example-folder
> cordova platform add android
> cordova build
> cordova emulate
> cordova run android
```

## Directory structure

### Overrides

## But I don't want to setup my environment, or I don't own a Mac but I need to make an iPhone version

No worries, Adobe got a very nice service called [PhoneGap Build]
which is targeted at solving just that problem. You write your app
locally and test it out in your local browser and when you feel
satisfied and want to test it on your device you package it up and
send it to PhoneGap build to be built. Then you can install it from
there onto your devices.

It actually works great and I'm using it for all my projects for
Android and iOS, I haven't had much luck with BB10 and WP8 although
WP8 support might be good now.

PhoneGap Build is inexpensive and it comes with what is referred to
as [hydrated apps] which is a quick way to update your apps on your
devices during development. They also got an awesome remote debugger
called [weinre], WebInspector Remote, which gives you access to the DOM
in an interface very similar to Chrome's web inspector.

The only downside with PhoneGap Build's debugger is that it's really,
really slow in Singapore. So for that I recommend installing your own
weinre server and setting the configuration points in the `config.xml`
for your project.

### weinre support

Android, BlackBerry 10 and iOS got great weinre support and I got
nothing bad to say about those platforms. Which leads us to...

#### Windows Phone 8

I've been spending a fair amount of time with another developer on IRC
trying to figure out how to get weinre properly working with PhoneGap
for WP8 and we've not been overly successful. Lack of interest in the
Windows platform as well as time since the project ended haven't made
great strides from me to try and fix WP8 and weinre. And from the
complete lack of movement for [my ticket][wp8-weinre-ticket] on the
Cordova bugtracker it seems they feel the same way. :)

#### BlackBerry 10

BlackBerry 10 has no problems at all, although they have their own
debugger running which is better so you should be using that one
intead. It's available on port 1337 when you're connected.

### A word about code signing

Depending on the platforms you're targeting the code signing will hit
you, for Android when you're ready to send it off to the Market. For
iOS the first time you want to run the app on your device.

Make sure you read the documentation for each platform so you
understand what is required of you when it comes to your code signing.

With that said here's some points for Android and iOS.

#### Android

The signing key you generate for Android is the only one that will be
able to create a signed app for you. Loose that key and you can't sign
that app anymore.

#### iOS
Apple has chosen a technically beautiful solution for handling its
apps. You create a x509 certificate signing request that Apple
signs. Then you create a provisioning profile for your app to which
you bind all the devices you want to use for testing. Truly beautiful
way to handle the problem of making sure people don't do something
they shouldn't.

It's a [PITA] to setup.

Read through the documentation about [iOS code signing][ios-signing]
and make sure you got everything correct. And if you're using the push
notification plugins you *need* to be using a distribution ad-hoc
certificate, else it will not work with PhoneGap Build.

# Writing your app

Use a reasonable framework that keeps you productive, I've used one
called Lavaca because it had all the features I wanted and it was
built to work with Cordova.

I do have a side project and for that one I'll try out [Backbone]
with [React] and see how that feels.

## It's just a normal web app, compress it

## Testing

Manual + logic in Jasmine. I haven't personally used any Selenium
tests but I'm sure they'd be awesome in here as long as you can get it
to run on your phones.

## WP8 quirks

* Network not available until after the `deviceloaded` event
* weinre doesn't work if `history.replaceState` is used
* css transforms didn't work
* svgs didn't load, although they are reported as working
* microsoft pointer events
* the fact you're stuck with internet explorer

# What did we learn?

Our original planning for the project was one week for getting the
basic app setup as a web app. Then one week per target platform for
tweaking and tuning.

This worked out pretty well for Android, iOS and BlackBerry. Windows
Phone 8 was a different story. Oh dear god was it a different story.

Take aways:

* Always make sure you got the time to do at minimum two submissions
  to the app stores that requires manual approval. Hopefully you won't
  need it but you may. Luckily we already planned for this.
* Don't assume anything when it comes to compatability, a checkbox
  saying it works is just a checkbox. Not that it actually works.
* Windows is not for developers or for when you want to get work done
  (oh okay, I might be more than just a little biased on this one).
* BlackBerry 10 is actually quite neat, I kind of wish it was a bigger
  platform so I could use the Z10.
* Android screen sizes are annoying, but I think we all know this
* Stay away from Windows Phone 8

## Windows Phone 8

So the initial time alloted for the project was six weeks, with one
week extra for unforseens. We ended up spending eight weeks with about
half of it spent on trying to get Windows Phone 8 to work decently.

We then had to go through submitting the app to the market place 3
times because WP8 injects permissions to the apps installed through
Visual Studio compared to apps installed throught he app loader. So
when I built the production build it didn't get the extra injected
setting that enabled remote networking and my app was rejected with
the unhelpful message from support, "The app doesn't load". Which it
clearly did.

This is also a point where it's worth mentioning that Microsoft got a
special test section on their store, where you can access the app the
same way that the technicians testing your app does. You create a beta
version of your app and through that you can then try out different
builds yourself the same way they do before you submit it. It's not
overly fast and between submissions you've to wait up to two hours for
the beta build to propagate before you can access it. But it sure
beats waiting five days for Microsoft to report back to you.

This last one is obviously on me for missing this section in their
documentation. The injecting hidden permissions I put solely on
Microsoft because it took quite a bit of searching to find it, and
Cordova has now [changed the default][wp8-net-permission] to always
give the net permission on WP8 projects to avoid this.


## Casualties one desk...

And at this point you all might be wondering about the title of this
talk? Well, first off, I work with way too much marketing people so
needing to have a catchy name has caught on.

It comes down to the hours before I finally managed to get WP8 to
start running my app consistently. I was working and my colleague knew
I was very frustrated and all of a sudden he heard a loud bang and
messaged me and asked if that was me making raging at my desk.

- Me? No. That was my desk repeatedly jumping up into my fist, I've no
  idea why it's doing that.
  
My desk is fine even though the white paint has some scuff marks I've
covered with my mouse pad. ;)



[Appcelerator Titanium]: http://www.appcelerator.com/
[RhoMobile]: http://docs.rhomobile.com/
[Cholo De Villa]: http://cholodevilla.com/#home
[James Rivett-Carnac]: http://github.com/yarbelk
[Björn Andersson]: http://sanitarium.se/
[platform support]: http://cordova.apache.org/docs/en/3.3.0/guide_support_index.md.html#Platform%20Support
[cordova-plugins]: http://plugins.cordova.io/
[push plugin]: https://build.phonegap.com/plugins/324
[PhoneGap Build]: https://build.phonegap.com/
[pointer events]: https://developer.mozilla.org/en-US/docs/Web/CSS/pointer-events
[backface-visibility]: https://developer.mozilla.org/en-US/docs/Web/CSS/backface-visibility
[backface-visibility-example]: http://jsfiddle.net/2mFz6/
[NODEJS]: http://nodejs.org
[CLI]: http://en.wikipedia.org/wiki/Command-line_interface
[hydrated apps]: http://docs.build.phonegap.com/en_US/2.9.0/tools_hydration.md.html#Hydration
[wp8-weinre-ticket]: https://issues.apache.org/jira/browse/CB-5219
[PITA]: http://www.urbandictionary.com/define.php?term=PITA
[ios-signing]: http://docs.build.phonegap.com/en_US/3.1.0/signing_signing-ios.md.html#iOS%20Signing
[wp8-net-permission]: https://issues.apache.org/jira/browse/CB-5354
