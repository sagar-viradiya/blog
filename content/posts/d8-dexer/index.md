---
title: "The D8 Dexer"
summary: New faster and optimised compiler
date: 2018-04-02
weight: 1
tags: ["Android", "Compilation", "Tool Chain"]
cover:
  image: images/header.jpeg 
  caption: "[Image source](https://i.ytimg.com/vi/d-zKJCKsoWw/maxresdefault.jpg)"
  hiddenInList: false
---

As an android dev we all are in concern with our app build time and app size. Faster build means more productivity and smaller APK size benefits your user and your product. That’s why we are always on the hunt of ways to optimize our build time as much as we can. For example, with the release of gradle plugin 3.0 one can leverage significant performance improvements to large multi-module projects.

However, there are certain steps in build process which contributes to build time and developer can’t do anything to optimize it. Dex compilation (Dexing) is one such process which mostly works under the hood in your day-to-day app development, but it directly impacts your app’s build time, .dex file size, and runtime performance.

> *For those who don’t know what is **dex compilation** ?
Dex compilation is a process of transforming .class bytecode into .dex bytecode for the Android Runtime (ART) (or Dalvik Virtual Machine for older version of android).*

Thanks to android tools team to come up with next-generation dex compiler, D8. D8 is the replacement for DX and it comes with new release of Android Studio 3.1.0 (Also available in studio 3.0 but disabled by default). D8 offers many advantages but before diving into D8 advantages let’s step back and explore default toolchain with desugaring to better understand all benefits D8 offers.

Default toolchain with desugaring
Remember [Jack toolchain](https://source.android.com/setup/build/jack) (deprecated now)? Which got introduced as a replacement of default toolchain to support java 8 features on android. But jack got deprecated due to the fact that it coverts your source code to .dex directly with no bytecode in between and developers relying on bytecode tools couldn’t use it. Besides it was slow in compilation.

Fast forward to today, android’s default toolchain provides built-in support for certain Java 8 language features using concept of desugaring. Now you may ask what is desugaring ?

{{< iframe src="https://giphy.com/embed/GYdyfzzDfanrq" width="480" height="259" frameBorder="0" class="giphy-embed" >}}

As you can see in the figure below it is a process which coverts java 8 bytecode generated by javac to java 7 compatible bytecode. This is the secret behind supporting certain java 8 features on all android versions by default toolchain.

{{< figure src="images/bytecode_transformations.webp" caption="[Java 8 language feature support using](https://developer.android.com/studio/write/java8-support.html) `desugar` [bytecode transformations](https://developer.android.com/studio/write/java8-support.html)." >}}

Okay so enough theory, let’s see one example of desugaring to make it clear.

Lambda is one of the popular java 8 feature and all devs love it because it makes your code easy to read by reducing boilerplate code. Since android runtime (ART or dalvik) doesn’t support java 8 natively it won’t be able to recognise lambda. Desugaring converts lambda expression into an anonymous class which is java 7 compatible.

So you can write click listener on the button using java 8 lambda feature like this

```java
button.setOnClickListener(v -> Log.d("Button clicked"));
```

which gets desugared to something like this

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.d("Button clicked");
    }
});
```

For more information on default toolchain head over to the [this](https://developer.android.com/studio/write/java8-support.html) page.

Okay, now we know enough about default toolchain so let’s explore advantages of D8 dexer.

## Advantages of D8 dexer

As I said earlier D8 is a new dex compiler which is integrated with gradle plugin 3.1.0. D8 is faster compare to DX thereby reducing build time and also it generates small .dex file thereby reducing your APK size. Also, it generates optimized .dex which gives you better run time performance.

Figures below show dex compilation time and dex file size comparison with DX.

{{< figure src="images/compile_time_comparison.webp" caption="[Compile time comparison](https://android-developers.googleblog.com/2017/08/next-generation-dex-compiler-now-in.html)" >}}

> Tested with benchmark project [here](https://github.com/jmslau/perf-android-large/tree/android-30).

{{< figure src="images/size_comparison.webp" caption="[DEX size comparison](https://android-developers.googleblog.com/2017/08/next-generation-dex-compiler-now-in.html)" >}}

> Tested with benchmark project [here](https://github.com/jmslau/perf-android-large/tree/android-30).

To take advantage of D8 all you need to do is to update your Android Studio to latest version 3.1.0.

You can further optimize your build time using D8 experimental feature called integrated desugaring. We have seen in default toolchain, desugaring is a separate process which runs immediately after java compilation as shown below.

{{< figure src="images/default_toolchain_desugaring.webp" caption="Default toolchain desugaring" >}}

With new experimental integrated desugaring it will be a part of D8 which will reduce your build time further.

{{< figure src="images/d8_integrated_desugaring.webp" caption="D8 integrated desugaring" >}}

Since it is an experimental feature you have to enable it by adding the following line in your gradle.properties file.

```groovy
android.enableD8.desugaring = true
```

Once it is stable it will be enabled by default in future Android Studio release.

## What’s next ?

Besides D8 dexer android tools team is also working on R8 shrinker, which is a proguard replacement for whole program minification and optimization. Think of R8 as D8 on steroids. It will convert your java bytecode to .dex by removing unused code and obfuscating it just like proguard.

Currently, it is not integrated with android gradle plugin but those who are curious about R8 can head over to open-sourced R8 project [here](https://android.googlesource.com/platform/external/r8/).

Finally, I would like to close this with a resource which will help you to understand D8 and R8 better. Tune into following podcast where [@chethaase](https://twitter.com/chethaase) and [@tornorbye](https://twitter.com/tornorbye) talk with [Jeffrey van Gogh](https://twitter.com/jvgogh) from the Tools team who works on D8 and R8.

[Episode 86: It's gr8!](http://androidbackstage.blogspot.in/2018/01/episode-86-its-gr8.html)

In case if you have any question or suggestion you can comment below or get in touch with me on [twitter](https://twitter.com/viradiya_sagar).

That’s it for now folks :)

> *This was originally posted on [Medium](https://medium.com/proandroiddev/the-d8-dexer-6736deb55fb8)*