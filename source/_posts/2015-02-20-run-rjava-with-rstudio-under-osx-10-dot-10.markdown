---
layout: post
title: "Run rJava with RStudio under OSX 10.10"
date: 2015-02-20 14:30:10 +0100
comments: true
categories: RStudio R OSX
published: true
---

# The Problem

I've had some trouble getting rJava to run with RStudio. My current solution is a small workaround based on [this post on the RStudio support pages](https://support.rstudio.com/hc/communities/public/questions/203781666-rJava-not-loading-in-RStudio-Mac-OS-X-10-10-but-loading-in-terminal).

The first thing I did was installing and running `rJava` through `RStudio` and attempt to load it.

{% codeblock lang:R %}
install.packages('rJava')
# a bunch of output follows
# ...
# Now we want to load rJava, which yields an error
library(rJava)
Error : .onLoad failed in loadNamespace() for 'rJava', details:
  call: dyn.load(file, DLLpath = DLLpath, ...)
  error: unable to load shared object '/usr/local/Cellar/r/3.1.2_1/R.framework/Versions/3.1/Resources/library/rJava/libs/rJava.so':
  dlopen(/usr/local/Cellar/r/3.1.2_1/R.framework/Versions/3.1/Resources/library/rJava/libs/rJava.so, 6): Library not loaded: @rpath/libjvm.dylib
  Referenced from: /usr/local/Cellar/r/3.1.2_1/R.framework/Versions/3.1/Resources/library/rJava/libs/rJava.so
  Reason: image not found
Error: package or namespace load failed for ‘rJava’
{% endcodeblock %}

When I've ran `R` through the Terminal and ran up `library(rJava)`, everything worked just fine. Somehow, `RStudio` is not able to properly load the java path. A small workaround helps here.

# The Workaround

We will launch `RStudio` through the terminal with a custom call, which gives it the proper `Java` path!

## Check if everything works as expected

At first we check a couple of things.

{% codeblock lang:bash %}
# `which java` should yield something like this: `/usr/bin/java`
$ which java
/usr/bin/java
# `/usr/libexec/java_home` should yield the current jdk home folder
$ /usr/libexec/java_home
/Library/Java/JavaVirtualMachines/jdk1.8.0_31.jdk/Contents/Home
{% endcodeblock %}

By the way, you can also use [`/usr/libexec/java_home`](https://stackoverflow.com/questions/21964709/how-to-change-default-java-version) to [switch your java version](https://stackoverflow.com/questions/21964709/how-to-change-default-java-version) if you wish. You can also download the newest `jre` from the Oracle [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

If the output is similar to the one above you should be fine!

## Run `RStudio` through the Terminal using a custom command

Now you open `RStudio` through the terminal using the following command, which sets the `LD_LIBRARY_PATH` by hard. *Make sure you quit all running instances of `RStudio` before you do that!*

{% codeblock lang:bash %}
$ LD_LIBRARY_PATH=$(/usr/libexec/java_home)/jre/lib/server: open -a RStudio
{% endcodeblock %}

`RStudio` launches. Loading `library(rJava)` should work just fine! *If you want to use `rJava` with `RStudio`, you have to run it using this command (until they hopefully fix it!)*

## Make an Alias for the new Command with your Shell

If you want you can register a alias with your shell. Using `bash`, I've opened my bash profile under `~/.bash_profile` and added the following alias:

{% codeblock lang:bash %}
alias rstudio='LD_LIBRARY_PATH=$(/usr/libexec/java_home)/jre/lib/server: open -a RStudio'
{% endcodeblock %}

# Thats it
I hope this gets fixed soon. Until then, hopefully my workaround works for you guys! See you :).
