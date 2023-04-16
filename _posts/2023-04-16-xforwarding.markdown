---
layout: post
title:  "Xfowarding Hyper-V VMs in Windows"
date:   2023-04-16 14:40:16 +1200
categories: linux
---

## XForwarding gnu/linux applications into your Windows environment

![XForwarded Desktop](https://raw.githubusercontent.com/xtenduke/xtenduke.github.io/dd981bf1f297b5b98b977b897b7af6de6335971a/assets/images/xforward.png "Xforwarded desktop")

## WSLg
Microsoft's WSLg really impressed me it gives you the ability to run GPU accelerated graphical linux applications in Windows through a RDP over a HV socket. While WSL(2) and WSLg are super cool pieces of cobbled together tech (yeah I love jank), the drawbacks I have found in WSL far outweighed the convenience of being able to launch graphical applications from windows.

![WSLg](https://raw.githubusercontent.com/xtenduke/xtenduke.github.io/c6902a08b838aaa54242272d6c187a26abaea585/assets/images/wsl-wayland.png "WSLg")


## Why not WSLg

- No "good" way to pass through USB devices. Forced to use USB over IP
- Performance issues
- Networking weirdness
- Slow filesystem

## Ditching WSL
I expect these issues to be resolved in the future, but in the meantime, due to these drawbacks, I ended up running linux VMs in Hyper-V, accessing them over SSH.
This gives almost native performance, Hyper-v is a type 1 hypervisor, in contrast to other offerings like Virutalbox (yuck, Oracle), or VMware Workstation ($$).

## Graphical applications
The main drawback of running linux in Hyper-V is the terrible graphical performance in the rare case that I need to run a gui app. Tools like the Remote extension for VS Code, and Jetbrains Gateway give you the ability to have a Windows gpu accelerated IDE, which is really sending instructions to a remote machine over SSH (my linux VM in this case).

But what if you want to run other GUI applications?


## XFowarding into windows
There are a few ways to do this, the best way I have found is to use a piece of commercial software called [X410](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi0yvui3K3-AhVA-jgGHTiKB2IQFnoECAwQAQ&url=https%3A%2F%2Fx410.dev%2F&usg=AOvVaw0pWaga9LvxlM_MbiFptB1h)

Setup is reasonably straight forward, I followed the VSock [guide](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi0yvui3K3-AhVA-jgGHTiKB2IQFnoECAwQAQ&url=https%3A%2F%2Fx410.dev%2F&usg=AOvVaw0pWaga9LvxlM_MbiFptB1h) as connecting over a Hyper-V specific mechanism seemed "nicer" to me than exposing another endpoint on my machine.


## XFowarding Issues
### Gnome
Gnome often had issues starting, I figured there was some incompatibility with the X410 xserver implementation. Since i'm not really using the Gnome desktop environment, just windows management, XFCE4 is a much better option.

### Launching applications... sucks

Unlike WSLg, there is no nice Windows explorer integration to launch apps, to get around this you could run an application launcher like Rofi or similar. Since I will always have a terminal window open, I just wrote a script that detaches commands you type, and sends output to /dev/null

![Janky application launcher](https://raw.githubusercontent.com/xtenduke/xtenduke.github.io/dd981bf1f297b5b98b977b897b7af6de6335971a/assets/images/launcher.png "Janky application launcher")

```
#!/bin/bash

run () {
    echo "Enter command:"
    read command
    exec $command </dev/null &>/dev/null &
    run
}

run
```


## Hardware acceleration
Graphics performance is fine for web browsing and other non-demanding tasks. Unfortunately, there is no direct access to the GPU, this means no GPU accelerated ML tasks, or native video encode/decode.


## What's next?
I am investigating the possibility of replicating Microsofts WSLg implementation on Hyper-V VMs, stay tuned.

