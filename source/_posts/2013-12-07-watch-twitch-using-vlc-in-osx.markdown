---
layout: post
title: "Watch Twitch using VLC in OS X"
date: 2013-12-07 00:20:42 +0100
comments: true
categories: [Twitch, VLC, OSX, streaming, Homebrew, livestreamer, rtmpdump]
---

If you own a Retina-Macbook you problably struggle with fairly high CPU load and bad performance when watching Twitch-channels using the standard flash-based player in the browser.

A tool called `livestreamer` can be used to bring Twitch streams to the beloved VLC player (which also uses the GPU to process videos). This way you are not just able to reduce the used resources, the streams also feel much smoother, especially for high resolutions.

*Twitch will not be able to stream ads if you use this solution.* 
*Please be fair and subscribe to channels you like and you support.*

Install Livestreamer
--------------------

- Download and install `rtmpdump` from [here](http://trick77.com/wp-content/uploads/2008/01/rtmpdump-2.4_mac_os.zip)
- Download `python-setuptools` from [here](https://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg#md5=fe1f997bc722265116870bc7919059ea)
- Open `Terminal.app`
- Navigate to the folder where you downloaded the Egg-File and install it
{% codeblock lang:bash %}
cd ~/Downloads
sh setuptools-0.6c11-py2.7.egg
# Maybe you need to run it as sudo - `sudo sh setuptools-0.6c11-py2.7.egg`
{% endcodeblock %}
- Clone `Livestreamer` GIT Repository and install `Livestreamer`
{% codeblock lang:bash %}
git clone git://github.com/chrippa/livestreamer.git
cd livestreamer
python setup.py install
# Again, maybe you need to run it as sudo - `sudo python setup.py install`
{% endcodeblock %}

Now you should be able to view Twitch channels in VLC

Using Livestreamer
------------------

Say you want to view this Twitch channel in VLC: [http://www.twitch.tv/wcs_europe](http://www.twitch.tv/wcs_europe).
All you have to to is go into the `Terminal.app` and type
{% codeblock lang:bash %}
livestreamer http://www.twitch.tv/wcs_europe [quality]
# You might as well skip the `http://www.` part
{% endcodeblock %}
Here, `[quality]` has to be a quality setting from the stream, usually ranging between `low`, `medium`, `high` and `source`. If you leave it empty, `livestreamer` will tell you which options you can choose from.

Et voilá. Enjoy your stream.
{% img center /media/2013-12-07-watch-twitch-using-vlc-in-osx/twitch-vlc-sc2_small.png %}

References
==========
Links and instructions how to install everything is taken from different Posts in this [Gamespot Thread](http://forum.gamesports.net/dota/showthread.php?45027-How-to-watch-Twitch-TV-in-VLC-player-(MAC-OSX-HOW-TO)