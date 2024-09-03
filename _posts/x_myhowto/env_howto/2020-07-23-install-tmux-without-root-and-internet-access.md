---
layout: default
title: Install tmux without root and Internet access
parent: Environment How to
grand_parent: My How to
description: Step by step to install tmux for linux user
comments: true
tags: [linux_env,cli_app]
nav_order: 1
toc_en: true
---

# Install tmux with no root and Internet
No Internet access in 2020?? Well, if you're working in this industry then you will not suprise about this. There is no way we can access to the Internet from the login server, and also not root at all. However we still can install some of our beloved tools without asking IT team.
{: .fs-5 .fw-500 }

---
## What do we need first
You should download these source files below to your local laptop and then copy them to your home dir in the linux server. 
<div class="code-example" markdown="1" >
   * libevent: [https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz](https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz)
   * ncurses: [ftp://ftp.gnu.org/gnu/ncurses/ncurses-5.9.tar.gz](ftp://ftp.gnu.org/gnu/ncurses/ncurses-5.9.tar.gz)
   * tmux: [https://github.com/tmux/tmux/releases/download/3.3a/tmux-3.3a.tar.gz](https://github.com/tmux/tmux/releases/download/3.3a/tmux-3.3a.tar.gz)
</div>
libevent and ncurses are two libraries that required for tmux. Your linux env might has these 2 libraries already, but there is high possibility that those libs are too old and incompatible.

---
## Then we install them locally
### First create your local directory
<div class="code" markdown="1" >
{% highlight sh %}
   mkdir -p $HOME/local
{% endhighlight %}
</div>
---
### Secondly install libevent
<div class="code" markdown="1" >
{% highlight bash %}
   tar xvzf libevent-2.0.19-stable.tar.gz
   cd libevent-2.1.12-stable.tar.gz
   ./configure --prefix=$HOME/local --disable-shared
   make
   make install
{% endhighlight %}
</div>
The `--prefix=$HOME/local ` is commonly used when installing a tool without root. Basically it will guide the compiling step to store any data to the `$HOME/local`. So remember this for later use.

---
### Then install ncurses
<div class ="code" markdown="1" >
{% highlight bash %}
   tar xvzf ncurses-5.9.tar.gz
   cd ncurses-5.9
   ./configure --prefix=$HOME/local
   make
   make install
{% endhighlight %}
</div>
---
### Finally install tmux
<div class ="code" markdown="1" >
{% highlight bash %}
   tar xvzf tmux-3.3a.tar.gz
   cd tmux-3.3a
{% endhighlight %}
</div>
Create a bash script, then copy this code below to that bash script (not tcsh or csh script), for example `compiletmux.sh`

<div class ="code" markdown="1" >
{% highlight bash %}
#!/bin/bash
 ./configure CFLAGS="-I$HOME/local/include -I$HOME/local/include/ncurses" LDFLAGS="-L$HOME/local/lib -L$HOME/local/include/ncurses -L$HOME/local/include"
 CPPFLAGS="-I$HOME/local/include -I$HOME/local/include/ncurses" LDFLAGS="-static -L$HOME/local/include -L$HOME/local/include/ncurses -L$HOME/local/lib" make
{% endhighlight %}
</div>
Then run this new script `compiletmux.sh`, you'll see a `tmux` binary file in current directory. 
Finally, copy this tmux to `$HOME/local/bin` and enjoy your new life with tmux.
