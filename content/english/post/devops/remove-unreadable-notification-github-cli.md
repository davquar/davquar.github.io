---
author: Davide Quaranta
title: Remove unreadable GitHub notifications
date: 2023-01-05T10:00:00+02:00
categories: [DevOps]
tags: [github, cli]
description: "This little post describes how to remove a special edge case of GitHub notifications that cannot be marked as read from the web interface"
---

I had a problem with GitHub: years ago I starred a repository that remained silent for some time; recently the authors decided to catch the attention of everyone and wrote a bot to mention every stargazer in a GitHub issue, promoting the project.

Eventually the repository went private (got banned?).

The problem is that I still had the **mention indicator** in my GitHub notifications. There is no way to remove it from the web interface.

Eventually I managed the remove the notification by using the GitHub CLI.

## Mark read a notification with GitHub CLI

1. Install the [GitHub CLI](https://cli.github.com/).
2. Authenticate from GitHub CLI.
3. Use `gh api notifications` to get a JSON containing all your notifications.
4. Note the notification (thread) ID (`<id>`) of the bad repository.
5. Execute `gh api --method PATCH notifications/threads/<id>` to *read* all the notifications of the given thread ID.

Use the [documentation](https://docs.github.com/en/rest/activity/notifications?apiVersion=2022-11-28) as reference.