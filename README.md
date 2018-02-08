HTPC Setup Guide
======
### What is this?
I got tired of retracing steps and rewriting config files every time I wanted to try out a new linux distro on my HTPC, so I'm documenting it this time. This is mainly for personal reference, but maybe someone else will find it useful. 
### What do I need?
This guide will build a capable HTPC from scratch, starting with a brand new linux installation. I will focus on **what I did**, which means:
- A desktop or laptop PC. If you're using something like a Raspberry Pi, I suggest you install [LibreElec](https://libreelec.tv/) and call it a day, as this setup will likely be overkill.
- An Arch-based Linux distribution. If you're running Ubuntu or another Debian variant, replace `pacman` commands with the `apt` equivalent and you should be fine.
- Init scripts will assume you're using `systemd`. If you're using Linux in 2018 and you don't know what `systemd` is, you're probably using it. If you're *avoiding* systemd in 2018, I'll assume that you have a very good reason for doing so, and are more than capable of writing your own initscripts.

Steps
=======

1. **Install Linux OS of choice.** I'm currently using [Antergos](https://antergos.com/), which has a very straight-forward installer. If you're also going with Antergos, make sure to tick the box for *AUR support* during setup to save you from having to setup `yaourt` later.

2. **Prepare dependencies and general tools.** Most of this is optional, but you'll want to install Kodi at this point, at a minimum. `x11vnc` will handle your remote desktop, and `samba` allows for filesharing with other devices on the network.

- `pacman -Sy kodi x11vnc samba git curl wget zsh binutils make` 

2a. *Totally optional and not HTPC-related in any way:*
- Visit [Oh My Zsh](https://ohmyz.sh/) and copy and paste one of their install strings. Either will work if you grabbed `git`, `zsh`, and either `curl` or `wget` in the previous steps. At the time of writing this, the following works: `sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
- `yaourt vtop` for a fancy terminal-based process manager (make sure to select vtop, **not** nvtop)
- `yaourt neofetch` for fancy screenshots

3. **Set up VNC server for remote desktop sharing.** I use my HTPC with a projector, meaning it's a huge pain in the ass to turn it on and do maintenance in my living room. VNC allows me to easily control the HTPC over the network.
- Choose a password for VNC logins `x11vnc -storepasswd /etc/x11.pass`
- Create or download [this init script](files/etc/systemd/system/x11vnc.service) to your systemd services path: `sudo wget -P /etc/systemd/system/ https://raw.githubusercontent.com/ababyduck/htpc_setup_guide/master/files/etc/systemd/system/x11vnc.service`
- `sudo systemctl enable x11vnc.service`
- `sudo systemctl daemon-reload`

3a. *Optional:* Reboot and login via VNC before proceeding. I recommend the [RealVNC client](https://www.realvnc.com/en/connect/download/viewer/) for Windows.

4. **Set up mount points with fstab.** If you're not using a separate NAS, you'll want to make sure your storage drives stay mounted.
- Create mount points. I store mine in /media/Movies/ and /media/TV/, but you can do whatever you like. e.g. `sudo mkdir -p /media/Movies/`
- Find the UUID's of your storage drives with `-lsblk -f`
- Append them to your `/etc/fstab` file and restart. See the last few lines of [my fstab](files/etc/fstab) for an example.

5. **Prepare Kodi for playback.** Kodi is great, but it has a ton of options and they're not always organized very intuitively.

-- IN PROGRESS--
