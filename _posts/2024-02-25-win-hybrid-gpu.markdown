---
layout: post
title:  "Windows hybrid GPU management"
date:   2024-02-25 00:00:00 +0800
categories: linux
---

Nvidia hybrid gpu setups in linux are incredibly painful due to poor driver support, nvidia drivers don't properly support being run as a secondary gpu, I could go on and on about this...


It seems that windows is not immune to hybrid gpu pains. 
I recently discovered failures in Windows 11 23H2 switching from dGPU to iGPU on my Lenovo P1 Gen 5 (intel Xe / nv 3070Ti).

### Behaviour
This machine is configured in a very common way, iGPU and dGPU are connected to the internal screen through a multiplexer, HDMI and USB-C ports are wired directly to the Nvidia GPU.

This means that the dGPU must be active to use external displays, but can be powered down for a portable configuration. The fact that the gpu can power down is really useful as even idle the gpu draws ~15w.

Typical behaviour in Windows for hybrid GPU setups is that the "High performance" gpu will be powered if either:

- A high-performance application is running, i.e. game
- An external display is plugged in

My experience has been that the dGPU doesn't actually power down when the external display is disconnected, resulting in terrible battery life after unplugging my machine from displays.

Based on information from Nvidia control panels "gpu activity icon" I have been able to deduce that the dGPU isn't powering down because there are applications running on it. To me this shows that it's a windows issue, surprise surprise...

### Monitoring
Open the Nvidia control panel - Desktop -> Display GPU Activity Icon in notification area.

![nvcp](https://github.com/xtenduke/xtenduke.github.io/blob/7e74a7d8c323a8f589b38ffa7a11085ec7e088e8/assets/images/nvidia-dgpu-cp-option.png)
![tray](https://github.com/xtenduke/xtenduke.github.io/blob/7e74a7d8c323a8f589b38ffa7a11085ec7e088e8/assets/images/nvidia-dgpu-tray.png)

### Solution
Disable and re-enable the nvidia dGPU when you undock.
The `battery.ps1` powershell script disables the nvidia GPU and re-enables it which effectively kicks all processes running on the dGPU off. Be warned that this may unsafely kill any applications you are using that depend on the dGPU, but i've had no issues with things like browsers etc, ymmv.

`battery.ps1`
```battery.ps1
Get-PnpDevice -Class "Display" | Where-Object Manufacturer -eq "NVIDIA" | Disable-PnpDevice -Confirm:$false
Get-PnpDevice -Class "Display" | Where-Object Manufacturer -eq "NVIDIA" | Enable-PnpDevice -Confirm:$false
```

I used this command prompt convenience script to call the PS script manually, but for long term use you probably want to automate this based on power events or display PNP events.

`battery.cmd`
```battery.cmd
powershell -ExecutionPolicy Bypass -File c:\Users\jake\Desktop\battery.ps1
```


### Result
'convincing' the dGPU to turn off changes this machine from having 1.5h battery life, to 3-4h (usage dependant), which I consider a massive win. Even if I am forced to run windows.
