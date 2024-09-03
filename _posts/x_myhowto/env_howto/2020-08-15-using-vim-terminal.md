---
layout: default
title: Using Vim terminal
parent: Environment How to
grand_parent: My How to
description: How a design verification engineer uses Vim terminal
comments: true
tags: [linux_env,vim,cli_app]
nav_order: 1
toc_en: true

---

# Using Vim terminal
Vim supports the terminal feature since version 8.1. This feature allows us to have a terminal running in a Vim window . I was confused at the beginning, but after a while using this terminal, I have to admit that it's pretty useful and handy in many cases such as test your code immediately on the terminal, yank and paste the data from terminal to vim, etc. Let's see how it works.
{: .fs-5 .fw-500 }

---
## Using vim terminal
### Open vim terminal
* First to use vim terminal, we should have vim from version 8.1 and compiled with `+terminal` option.
* To open vim terminal, just type the command `:terminal`, or `:vert terminal` to open the terminal in a vertical split window.
* To close the vim terminal, type `exit` or `Ctrl-W Ctrl-C`
* To move between buffers and vim terminal buffer, use `:buffers` as normal, since vim terminal will be treated as a buffer in vim. Or if current vim terminal is opened in another window, just simply use `Ctrl-W Ctrl-W`.

### Vim Terminal-Normal and Terminal-Job mode
Terminal-Normal mode and Terminal-Job mode are newly introduced along with vim terminal feature.
* Terminal-Job mode is the default mode when we open the vim terminal. It's actually will be like a normal terminal, where we can use the cli command.
* Terminal-Normal mode is kind of a cool mode that I really enjoy. It allows us to move the cursor around the terminal screen in a vim-way, and we also perform vim commands such as yanking, selecting text, but not modifying command like deleted, change, replace command.
* Type `Ctrl-W N` or `Ctrl\ Ctrl-N` to enter Terminal-Normal mode. Type `i` or `a` to return to Terminal-Job mode.
<script id="asciicast-353835" src="https://asciinema.org/a/353835.js" async></script>

---
## Vim send data from a buffer to vim terminal
### Using registers
To copy the data from the vim buffer to the vim terminal buffer, we can use registers. Simply yank the data from normal vim buffer to vim registers, then go to the vim terminal (in Terminal-Normal mode), then paste the data from the register using this command `Ctrl-W " {register}`.

### Using vim-sendtowindow
Wanna be a little bit faster, use this [vim-sendtowindow](https://github.com/KKPMW/vim-sendtowindow) plugin. This plugin actually doing the same thing as we discussed above but using vimscript: yank the selected data to a registers, then send the data in the register to the vim terminal. However I think it's quite handy when testing your code with vim terminal, check the plugin page if you're interested.

---
## Open another file in vim terminal
Sometimes when I navigating through files inside a vim terminal and I want to open a file right from the terminal, I end up like in this figure below. A file is opened inside vim terminal with 2 status lines, one for the new opened file, one for the vim terminal (means the status line of current opened vim). This is pretty annoying to me.
![vim terminal](https://8zuu0g.by.files.1drv.com/y4ms0VTF1SD3HMx67macafhAd9I7wHvT_mcW8zxEtabiMeFQHpkzte1YonNH3tE4GFSvdnoRfidzg96CxHchAKGBTOD2dlE7vEWciIe_vt_WAxC4Crva_Wci0mNzbBiuKUmswTg4me5_Z_-Y8I4gQLyUB7Xr_ZXD92PgxDEGEOXmpl9xIxxAl8njyzZ1RXGnEHMlpVG9Sylj0I-T4-LWZGwTg)

So I create this bash script, which is simply to open the new file from vim terminal to the current vim. Check `:h terminal` for more detail of how it works.
<script src="https://gist.github.com/dvtalk/7a1776737f4302b3ed7b71b68fce1b35.js"></script>
The command is `printf "<ESC>]51;[\"drop\", \"$pathdir\"]^G" ` but I do not figure out how to display them correctly in this post :D.

Then in your `.cshrc` or `.bashrc` file, create an alias for vim command as below (for tcsh) so that we can use this script to open file as normal vim when in vim terminal.
<div class ="code" markdown="1" >
{% highlight bash %}
# tcsh
if ( $?VIM_TERMINAL ) then
   alias vi vimsend
endif
{% endhighlight %}
</div>

Now let's see how it's like when using this script to open a file inside vim terminal
<script id="asciicast-354593" src="https://asciinema.org/a/354593.js" async></script>



## Conclusion
Those things above is pretty much everything that I've been doing inside vim terminal. If you like this feature already and want to dig deeper, just check `:h terminal`
