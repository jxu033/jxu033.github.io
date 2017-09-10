---
title: "How to Edit Your Path Environment Variables On Mac OS X"
layout: post
date: 2017-09-10 23:44
tag:
- Path Environment Varialbes
- MacOS X
star: true
category: blog
author: jiaqixu
description: Path Environment Variables On MacOS X
---
## How To Edit Your PATH Environment Variables On Mac OS X

If you are new to Mac OS X, you may need to know how to edit your PATH. The good news is that this is an easy task on Mac OS X.

The recommended way is by editing your .bash_profile file. This file is read and the commands in it executed by Bash every time you log in to the system. The best part is that this file is specific to your user so you won’t affect other users on the same system by changing it.

<b>Step 1:</b> Open up a Terminal window (this is in your Applications/Utilites folder by default)

<b>Step 2:</b> Enter the follow commands:

{% highlight raw %}
touch ~/.bash_profile; open ~/.bash_profile
{% endhighlight %}

This will open the .bash_profile file in Text Edit (the default text editor included on your system). The file allows you to customize the environment your user runs in.

<b>Step 3:</b> Add the following line to the end of the file adding whatever additional directory you want in your path:

{% highlight raw %}
export PATH="$HOME/.rbenv/bin:$PATH"
{% endhighlight %}

That example would add ~/.rbenv to the PATH. The $PATH part is important as it appends the existing PATH to preserve it in the new value.

<b>Step 4:</b> Save the .bash_profile file and Quit (Command + Q) Text Edit.

<b>Step 5:</b> Force the .bash_profile to execute. This loads the values immediately without having to reboot. In your Terminal window, run the following command.

{% highlight raw %}
source ~/.bash_profile
{% endhighlight %}

That’s it! Now you know how to edit the PATH on your Mac OS X computer system. You can confirm the new path by opening a new Terminal windows and running:

{% highlight raw %}
echo $PATH
{% endhighlight %}

You should now see the values you want in your PATH.