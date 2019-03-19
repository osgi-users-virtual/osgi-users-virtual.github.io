---
layout: post
title:  Debugging JLink'd Java Image
date:   2019-02-24 12:00:00
categories: general
tags: [general, jlink]
author:
    name: Raymond Augé
    email: rotty3000@gmail.com
    twitter: rotty3000
    github: rotty3000
---

Recently I've been playing at pairing OSGi development with Java's `jlink` mechanism. Small, fit for purpose applications seem to be the norm these days and since OSGi has all the qualities required to achieve this it seems natural to pursue this combination.

Progress was being made until I needed to debug an issue which only manifested *after* `jlink`.

<!-- more -->

## Update [Mar 19, 2019]

As it turns out all you need to do is make sure that your `jlink`'d modules include `jdk.jdwp.agent`. 

Once included, the argument:

```txt
-Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000
```

should work as expected.

Thanks to an industry colleague for instructing me on the correct way to do this. :) You know who you are!



##### Ignore everything bellow this line (leaving it for posterity only.)

------

## Attempt #1

My first instinct was simply too add the often used command line options to for debugging, usually something like:

```txt
-Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000
```

The entire command line looked like this (where `executable` is the root of the `jlink`'d image and the name of the custom module):

```sh
$] executable/bin/java -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000 -m executable
```

This produced the error:

```txt
Error occurred during initialization of VM
Could not find agent library jdwp on the library path, with error: libjdwp.so: cannot open shared object file: No such file or directory
```

## Attempt #2

Next, my queries to *the Google* didn't provide much in the way of concrete answers. I guess there aren't that many people who have been in this situation to this point, which I found to be odd. I'm not really sure how to explain this. I can only think of 3 possible reasons (I'll admit there could be many more):

1. people haven't found the need to debug their `jlink`'d applications
2. people haven't used `jlink`
3. it's so obvious how this should be done that documenting the process isn't useful information

Which do you think it is?

## Attempt #3

Ultimately, I had to dust off a latent skill called *reading the documentation*.

As it turns out (and if you think this obvious given we're using `jlink`, then you might be right) that the `jlink`'d image prunes off not only Java modules but also related native libraries which is perfectly fine. Semi-related to this the image doesn't contain `libjdwp` (the library that enables debugging).

I'm willing to accept this as a fact that my image didn't include `java.instrument` which is the API I'd guess most likely links the functionality of `libjdwp`. Sadly, even including `java.instrument` doesn't cause `libjdwp` to be included with the image which kind of breaks with symmetry.

Alternatively it would be nice if there was a `jlink` switch which allowed me to include particular libs as part of the build. I personally feel `libjdwp` is pretty critical in most cases.

Luckily help information provided by the `jlink`'d `java -h` command explains how to add a library which isn't found in the default library path of the image:

```
 -agentpath:<pathname>[=<options>]
                  load native agent library by full pathname
```

I suspected that I should use the library that matched the version of the JDK (`11`) I had used to build the `jlink` image, but for fun I tried to use a different version to see what would happen (note I'm on Linux and using [Azul Zulu](https://www.azul.com/downloads/zulu/) of which I have JDK 7 through 11 installed, so mixing and matching here was simple):

```sh
$] executable/bin/java -agentpath:/usr/lib/jvm/zulu-9-amd64/lib/libjdwp.so=transport=dt_socket,server=y,suspend=y,address=8000 -m executable
```

The result was:

```txt
ERROR: This jdwp native library will not work with this VM's version of JVMTI (11.0.0), it needs JVMTI 9.0[.0].
```

It's good to know that this type of protection is in place so that you don't end up having to debug the debug configuration :|.

## Eureka!!!

In the end the command that allowed debugging to finally work was:

```sh
$] executable/bin/java -agentpath:/usr/lib/jvm/zulu-11-amd64/lib/libjdwp.so=transport=dt_socket,server=y,suspend=y,address=8000 -m executable
```

## Conclusion and A Caveat

The exclusion of debugging support does feel like it might leave us susceptible to a maintenance issue where some time in the future we have a deployment to debug, but can't conveniently find the exact matching JDK libs, or they aren't deployed to the environment; leaving us to jump through hoops that may not be pleasant.

My suggestion would be to copy the two libraries necessary for debugging; `libjdwp.so` and `libdt_socket.so` into the image `lib` directory at build time:

```sh
$] cp /usr/lib/jvm/zulu-11-amd64/lib/libjdwp.so executable/lib/
$] cp /usr/lib/jvm/zulu-11-amd64/lib/libdt_socket.so executable/lib/
```

The debugging command line then becomes:

```sh
$] executable/bin/java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=8000 -m executable
```

Lastly, the space required by these two libraries (`271K` and `26K` respectively) I'd consider to be an insignificant price compared to the headache you probably end up finding yourself in in the future by not having included them in the image.