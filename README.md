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

2a. *100% optional:*
- Visit [Oh My Zsh](https://ohmyz.sh/) and copy and paste one of their install strings. Either will work if you grabbed `git`, `zsh`, and either `curl` or `wget` in the previous steps. At the time of writing this, the following works: `sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
- `nano ~/.zshrc` and change the ZSH theme from bobbyrussel to wedisagree. Save and type `source ~/.zshrc` to refresh.
- Customize your wallpaper and set a nice dark theme. In Antergos, open the Activities menu with your Windows Key, start typing "tweaks" and open up the Tweaks application. Under Appearance, tick Global Dark Theme, then set the application theme to Adwaita-dark. Then open up Terminal and go to Edit > Profile Preferences > Colors. Tick "Use colors from system theme" and set the color palette to Solarized.
- Install your favorite Chromium Extensions (I use [uBlock Origin](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm), [uBlock Origin Extra](https://chrome.google.com/webstore/detail/ublock-origin-extra/pgdnlhfefecpicbbihgmbmffkjpaplco), [Privacy Badger](https://chrome.google.com/webstore/detail/privacy-badger/pkehgijcmpdhfbdbbnkijodmdjhbjlgp), [Video Speed Controller](https://chrome.google.com/webstore/detail/video-speed-controller/nffaoalbilbmmfgbnbgppjihopabppdk), [Don't Fuck With Paste](https://chrome.google.com/webstore/detail/dont-fuck-with-paste/nkgllhigpcljnhoakjkgaieabnkmgdkb), and [Reddit Enhancement Suite](https://chrome.google.com/webstore/detail/reddit-enhancement-suite/kbmfpngjjgdllneeigpgjifpgocmfgmb) on every machine I use.)
- `yaourt vtop` for a fancy terminal-based process manager (make sure to select vtop, **not** nvtop)
- `yaourt neofetch` for fancy screenshots

3. **Set up remote desktop sharing with X11VNC.** I use my HTPC with a projector, meaning it's a huge pain in the ass to turn it on and do maintenance in my living room. VNC allows me to easily control the HTPC over the network.
- Choose a password for VNC logins `x11vnc -storepasswd /etc/x11.pass`
- Create or download [this init script](files/etc/systemd/system/x11vnc.service) to your systemd services path: `sudo wget -P /etc/systemd/system/ https://raw.githubusercontent.com/ababyduck/htpc_setup_guide/master/files/etc/systemd/system/x11vnc.service`
- `sudo systemctl enable x11vnc.service`
- `sudo systemctl daemon-reload`

3a. *Optional:* Reboot and login via VNC before proceeding. I recommend the [RealVNC client](https://www.realvnc.com/en/connect/download/viewer/) for Windows.

4. **Set up mount points with fstab.** If you're not using a separate NAS, you'll want to make sure your storage drives stay mounted in a static location.
- Create mount points. I store mine in /media/Movies/ and /media/TV/, but you can do whatever you like. e.g. `sudo mkdir -p /media/Movies/`
- Find the UUID's of your storage drives with `-lsblk -f`
- Append them to your `/etc/fstab` file and restart. See the last few lines of [my fstab](files/etc/fstab) for an example.

5. **Prepare Kodi for playback.** Kodi is great, but it has a ton of options and they're not always organized very intuitively.

- Launch Kodi
- Go to Settings (gear icon at the top) > Interface settings > Skin > Get more... > Arctic Zephyr
- Back out to Settings, then go to Addons > Install from repository > Program add-ons > Artwork Downloader. Install this
- Back out to Settings, then go to Skin Settings > Home > Customize Home Menu
- Remove all items except Movies, TV shows, Settings
- Add a fourth option, label it something like Menu, then go down to Submenu
- Add four new items to this menu, then "Choose item for menu" for each of them.
- The four items we want are Kodi Command > Update Video library, Kodi Command > Clean Video Library, Add-on > Program > Artwork Downloader, and Kodi Command > Quit
- Finally, we'll add our media. Back out to Settings again, then go to Library > Videos... > Add Videos
- I add sources for TV, Movies, Documentaries, Anime (Series), and Anime (Movies), but this may vary depending on how you organize your media. For movies, pay extra attention to the "Movies are in separate folders that match the movie title" option, or Kodi will arbitrarily assume that the folder you store all of your movies in is actually just one movie. TV shows don't behave this way, because consistency would make things too easy.
- Finally, back out to the main screen in Kodi. Go to Menu > Update Video Library, then Menu > Artwork Downloader. Both of these may take a while, depending on how many media files you've got stored. Once that's done, you should have a nice clean home screen with Movies, TV Shows, Settings, and our custom Menu, complete with art.
- Open up the Movies menu, hit the left arrow, then tick "Show plot" to show synapses for selected items.
- There's a whole lot more you can do in Kodi, but I'm pretty happy with this setup. Feel free to install add addons for streaming services or configure further.

6. **Set up network shares using Samba**

- Create /etc/samba/smb.conf, using [this file](files/etc/samba/smb.conf) as a template.
- `testparm` to make sure there are no typos in the config.
- `sudo systemctl enable smbd`
- `sudo systemctl enable smbd`

7. **Begin setting up automation services**

-- IN PROGRESS--
