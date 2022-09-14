---
author: Davide Quaranta
title: Replacing xfwm4 with Mutter
date: 2022-09-14T11:00:00+02:00
categories: [Linux]
tags: [desktop, xfce, gnome]
description: "This is an ongoing experiment. One day I've decided to stop using the default XFCE's window manager xfwm4 and start using Mutter instead, which is the one shipped with GNOME. Switching is easier than expected, but at the end there are a few usability issues, some of which are quickly solvable but others are trickier."
---

## Motivations

Why someone would want to ditch **xfwm4**? This is a list of some problems that I had with it, that made me think about switching to something else:

* **Window borders** are too thin, thus difficult to resize.
* **Screen tearing**.
* Compositors like **Compton/Picom** don't solve tearing.
* Some **theme** authors don't curate enough xfwm4 assets.

Take into account that I'm using a **nVidia** GPU.

## Steps

Switching is fairly easy.

### 1. Install Mutter

If you are on a Debian-based system like me:

```shell
sudo apt install mutter
```

Make sure that it is newer than 41.1, because otherwise you end up with a [broken session when switching to a new workspace](https://gitlab.gnome.org/GNOME/mutter/-/issues/2038).

### 2. Try it out

Before setting Mutter as default window manager, let's try it out:

```shell
mutter --replace
```

Some things that are good to try:

- Just do your normal work.
- Switch workspaces.
- Play with screen resolutions.
- If you have multiple monitors, play with their placement.
- Play with keyboard shortcuts.

### 3. Do some quality-of-life tweaks

The default configuration of Mutter is a little but funky, especially in this case when we don't have a full GNOME Shell.

We can tweak things a little.

#### Enable common-sense window buttons

To have a human-friendly window button configuration layout:

```shell
gsettings set org.gnome.desktop.wm.preferences button-layout ":minimize,maximize,close"
```

Feel free to use other dispositions if you prefer.

#### Remove the default overlay key

In this Frankenstein desktop situation, it's better to remove the default overlay key `Super`, otherwise it would prevent other shortcuts to work.

```shell
gsettings set org.gnome.mutter overlay-key ""
```

#### Tweak the Alt-Tab behavior

The windows switching behavior is the only **serious usability issue**. In its default state it only switches between the two last used windows, with no option to cycle at all.

This may work: (taken from the [Arch Wiki](https://wiki.archlinux.org/title/GNOME/Tips_and_tricks#Navigation)).

```shell
gsettings set org.gnome.desktop.wm.keybindings switch-applications "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-applications-backward "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-windows "['<Alt>Tab', '<Super>Tab']"
gsettings set org.gnome.desktop.wm.keybindings switch-windows-backward  "['<Alt><Shift>Tab', '<Super><Shift>Tab']"
```

In my case it wasn't good at all, so I used [Rofi](https://github.com/davatorium/rofi/) to create a custom windows switcher script, binded to the `Alt+Tab` shortcut.

If you want to explore that solution too, have a look at [this issue](https://github.com/davatorium/rofi/issues/38); I specifically tackled this problem with this Rofi launch configuration:

```shell
xdotool mousemove 2400 600 sleep 0.1 mousemove restore &
rofi -no-lazy-grab -show windowcd -modi windowcd -theme grustom-window-switcher.rasi -kb-accept-entry '!Alt-Tab,!Alt+Alt_L,Return' -kb-row-down 'Alt+Tab' -selected-row 1
```

The idea is simple:

* Rapidly move the mouse to the center of the screen (where Rofi shows), and restore its position.
* Show Rofi and allow the release of `Alt+Tab` to switch windows.

The mouse thing is needed because (as described in the issue) to accept the release of those keys, the mouse needs to be where Rofi is, at launch time.

If you don't like this approach, there are other window switchers to pick.

#### Install GNOME Control Center

To change GNOME-related settings more easily, we can install GNOME Control Center:

```shell
sudo apt install gnome-control-center
gnome-control-center
```

If you end up in a panel that makes the app crash, you need to reset its state, otherwise you the control center doesn't start anymore:

```shell
gsettings reset org.gnome.ControlCenter last-panel
```

### 4. Set Mutter as default window manager

If everything works and you are ready to **switch from xfwm4 to Mutter**:

```shell
xfconf-query -c xfce4-session -p /sessions/Failsafe/Client0_Command -t string -sa xfsettingsd
xfconf-query -c xfce4-session -p /sessions/Failsafe/Client1_Command -t string -sa mutter
```

More info on the [Arch Wiki](https://wiki.archlinux.org/title/xfce#Use_a_different_window_manager).

## Final thoughts:

* Switching is easy.
* The final result is good enough.
* Custom themes look beautiful.
* There are some usability issues.
	- Some of them can be easily solved.
	- The `Alt+Tab` issue is a pain.
		+ It can be solved with external windows switchers.