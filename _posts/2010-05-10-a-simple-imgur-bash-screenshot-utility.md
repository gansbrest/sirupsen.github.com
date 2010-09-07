---
layout: post
title: A simple Imgur Bash screenshot utility
---

I use screenshots a lot, every day. Mostly when I do instant messaging, they can usually help explain something much quicker than anything else. It's rare that I edit the screenshot, and in these rare occasions, it doesn't bother me all that much having to fire up [Pinta](http://pinta-project.com/) or [Gimp](http://www.gimp.org/) - to make these small changes.

## Coming up with the script

The functionality needed, came down to this:

* Select region and take screenshot of this region
* Upload screenshot to [Imgur](http://imgur.com)
* Put direct link to screenshot into the clipboard

Taking a screenshot, of a specified region is quite easy with `scrot`:

{% highlight bash %}
scrot -s
{% endhighlight %}

Then using `curl` to upload the picture, via the Imgur API:

{% highlight bash %}curl -s -F "image=@$1" -F "key=api-key" \
http://imgur.com/api/upload.xml {% endhighlight %}

This returns some XML containing, among other things, the direct URL to the uploaded screenshot, which we extract from the returned XML with a simple regex:

{% highlight bash %}grep -E -o "<original_image>(.)*</original_image>" | \
grep -E -o "http://i.imgur.com/[^<]*" {% endhighlight %}

Now we have the direct link, and then it's simply a matter of putting this all into the clipboard with `xclip`:

{% highlight bash %}xclip -selection c {% endhighlight %}

Now this is optional, but quite handy. It uses `libnotify` to notify you when the image is uploaded, and ready to be pasted:

{% highlight bash %}notify-send "Clipboard ready!" {% endhighlight %}

## The script

And I compiled all this into this simple script (I'm aware that this can be a one-liner, and everything, but this just seems more readable, and *works*. If you have a better solution, please contact me!):

{% highlight bash %}
function uploadImage { 
  curl -s -F "image=@$1" -F "key=486690f872c678126a2c09a9e196ce1b" [http://imgur.com/api/upload.xml][] | grep -E -o "<original\_image\>(.)\*</original\_image\>" | grep -E -o "<a >http://i.imgur.com/[\^</a><]\*" 
}

scrot -s "shot.png"
uploadImage "shot.png" | xclip -selection c
rm "shot.png"
notify-send "Done" function uploadImage { 
  curl -s -F "image=@$1" -F "key=486690f872c678126a2c09a9e196ce1b" [http://imgur.com/api/upload.xml][] | grep -E -o "<original\_image\>(.)\*</original\_image\>" | grep -E -o "<a >http://i.imgur.com/[\^</a><]\*" 
}

scrot -s "shot.png"
uploadImage "shot.png" | xclip -selection c
rm "shot.png"
notify-send "Done" 
{% endhighlight %}

## Installation

This script assumes that `~/bin` is in your path, if not issue the following first:

{% highlight bash %}
echo "PATH=$PATH:~/bin" >> ~/.bashrc
{% endhighlight %}

{% highlight bash %}
curl http://sirupsen.com/static/misc/shoot > ~/bin/shoot && chmod 755 ~/bin/shoot
{% endhighlight %}