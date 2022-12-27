---
author: Davide Quaranta
title: "TIL: SSH Jump Host"
date: 2022-11-15T12:00:00+02:00
categories: [DevOps]
tags: [til, linux, ssh]
description: "The jump host is an option of the SSH client, that allows to use a third SSH server as \"proxy\" to access the final intended SSH server"
series:
  - TIL
---

{{< notice info >}}
This post is part of the TIL (Today I Learned) series; they are small posts in which I write something that I learned recently, and found interesting enough to be shared in this format.
{{< /notice >}}

Last week I needed to access a remote server to do some maintenance but I couldn't connect via SSH. The problem was that I had configured the **SSH server** to listen to a **custom port** and the network I was connected to had a very strict **firewall** configuration that blocked outbound SSH on ports different than 22.

How could I **bypass** the firewall? Yes, I could SSH on another host on port 22 and then access my final destination from here.

But I didn't want to mess with SSH keys.

Then I discovered that this use case is actually considered by SSH: there is a simple option that allows to specify **jump hosts**, namely hosts that are in-between you and your final destination.

By specifying a jump host we realize a *proxy* behavior.

## Usage

Let's see the `man` pages:

```
[...]
-J destination
	Connect to the target host by first making a ssh connection to the jump host
	described by destination and then establishing a TCP forwarding to the
	ultimate destination from there.  Multiple jump hops may be
	specified separated by comma characters.  This is a shortcut to specify a
	ProxyJump configuration directive.  Note that configuration directives
	supplied on the command-line generally apply to the destination
	host and not any specified jump hosts.  Use ~/.ssh/config to specify
	configuration for jump hosts.
[...]
```

We can use jump hosts in this way:

```shell
ssh -J jump final
```

Where `jump` is the jump host and `final` is the final SSH server.

For example:

```shell
ssh -J user@myserveron22 user@myserveroncustomport
```

Since the only thing that the firewall sees is that I'm SSH-ing on port 22, it is allowed.