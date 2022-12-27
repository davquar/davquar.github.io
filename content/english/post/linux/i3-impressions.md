---
author: Davide Quaranta
title: "Using i3 for two months: my impressions"
date: 2022-12-27T13:00:00+02:00
categories: [Linux]
tags: [desktop, i3, window-manager]
description: "i3 is a tiling window manager, which means that the concept of \"window\" does not exist. Instead of windows, there are \"tiles\": generic partitions of the screen, where some content is drawn into. In this post I write my opinions on the key changes between stacking and tiling window managers."
---

I had been traditionally using normal (also called *stacking*) window managers: the ones with the nice buttons on the right (or left) to minimize, maximize, close. The ones that you normally use with a taskbar.

Then one day I have decided to try out a **tiling window manager**, because *why not*. I choose [i3](https://i3wm.org/) for some reason.

I didn't think I would stick with it, but after some weeks I find myself writing this post on a train, with a laptop configured with **i3**, [polybar](https://polybar.github.io/), and a bunch of little external utilities to recreate the little comforts of traditional desktop environments.

**Why?**

Here are some ideas. It is not a collection of _pros_ and _cons_, but just some thoughts on what changes between stacking and tiling window managers, focusing on my experience.

## Increase of productivity

I feel that with a tiling window manager productivity increases, mainly because:

- It is easier to be **focused** on a single thing.
- There is **less noise**/distraction on screen.

**Workspaces** allow for a **better organization** of your activities. There are multiple ways to do that, for example:

- Dedicating one workspace for a single task.
- Dedicating one workspace for a single application.
- Dedicatine one workspace for a single "class" of application.
- ...

You can choose the one that works better for you. I am currently organizing my workspaces per class of application: my workspaces are numbered from 1 to 9, and they are \[browsers, terminals, code, file managers, writing, IM, mail, music, other\]; workspace 0 is instead fixed on a **second screen**.

A good workspace management allows you to also be better at **multitasking**, just because you can have a lot more going on, **without all the graphical pollution** that you normally have with a stacking WM. 

Summarizing, tiling window managers enable a focused, clean and powerful environment.

## Decrease of productivity

In my experience, tiling window managers create a lot of room for **customization** and tweaking, so (especially at the beginning) you may spend a lot of time tweaking things instead of doing what you have to do.

Fighting with **keybindings** is also time-consuming. With traditional desktop environments you take for granted a lot of things that with tiling WMs you **have to** install and configure yourself.

Another issue is about **window switching**: `Alt+Tab` can sometimes be just easier and more natural than switching to a specific workspace or tile.

- `Alt+Tab`-ing frequent windows has a low **cognitive load** and is very easy in terms of **muscular memory**, since it is an easy keyboard combination.
- When there a multiple windows to cycle in `Alt+Tab`, the cognitive load increases, but the physical cost remains constant.

With **i3** you can have two different switching mechanisms instead of `Alt+Tab`:

- Moving to another tile on the same workspace.
- Switching to another workspace.

Depending on the keybinding that you have decided, this may be more or less costly than `Alt+Tab`. In my case, the `Win` key is the main modifier and I can switch tiles with the arrow keys, and workspaces with the associated number; for example:

- Move to the window on the right: `Win+→`.
- Move to workspace 3: `Win+3`.

With my configuration, the first case has a low cognitive and physical cost, but the second case costs a little bit more:

- You must remember that a particular window is on workspace `n`.
- It is more difficult to hit `Win+n` compared to `Alt+Tab`.

Summarizing, the decrease of productivity is driven by:

- Constant customization.
- Fighting with keybindings.
- Sometimes it is just faster to `Alt+Tab`.

## Workflow change

Undoubtedly tiling WMs introduce new workflows.

### Example: opening a code editor

1. `Win+2` to go in the terminals workspace.
2. cd to the desired directory.
3. `code .` to open VSCode in that directory.
4. Hit `Win+Shift+3` to move VSCode to workspace 3.
5. Hit `Win+3` to go in workspace 3.

Or you can just open VSCode from the `dmenu` or with an application launcher like Rofi, and move it to the dedicate workspace.

Or do something else. In any case, according to your *workspace policy* and preferences, you can do the same thing in many ways, and even **hard-assign** specific applications to specific workspaces.

### Example: dragging and dropping

Suppose that you want to upload a file on a web page:

1. Go to the web page.
2. Switch to the workspace in which you have the file explorer.
3. Start dragging the file to upload.
4. Switch to the browser workspace.
5. Release the file.

It is not difficult, unless you are using a **touchpad**.

## Conclusions

Tiling window managers offer the following over traditional ones:

- Focused environment.
- Better multi-tasking.
- More customization possibilities.
- More performance.

Even if, in some situations, having simple normal *windows* may be more comfortable.
In any case, I'm sticking with **i3**.