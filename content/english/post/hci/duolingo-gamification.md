---
author: Davide Quaranta
title: "How Duolingo uses Gamification to motivate the user"
date: 2022-09-23T17:00:00+02:00
categories: [HCI]
tags: [gamification, duolingo, ui, ux]
description: For the past month I've been daily using Duolingo to learn German, and I'm captured by the app's dynamics, which uses some gamification devices that first hook you into forming an habit, and then challenge and reward you for studying and learning more. I've then decided to pack some examples of the gamification tools that Duolingo uses and why.
toc: true
---

{{< notice info >}}
Some time after writing this post, Duolingo heavily restructured the app, so some pictures may be outdated; the concepts are still valid, though.
{{< /notice >}}

## What is Gamification?

**Gamification** can be defined as the act of incorporating typical **game features** in products/services that are not games.

A further definition of gamification is split in two levels:

* **Basic**: use direct devices like badges, scores, etc.
* **Advanced**: use meta-devices that naturally make people want to do something, in order to achieve a feeling of accomplishment.

For example, putting a banner on each step of a stairway, telling people how many cumulative calories they are burning  at each step, naturally creates a **game dynamic** that may encourage people to use the stairs instead of the escalator; this is an example of advanced gamification.

Conversely, rewarding people with a *Stairway coin* is a form of basic gamification.

## Gamification elements in Duolingo

Duolingo extensively uses **basic gamification** to motivate the  user and incentivize them to:

* Learn.
* Review past material.
* Test their memory and knowledge.
* Test their speed of recall.
* Create and maintain a regular learning habit.

The interesting thing about Duolingo is the intense **combination** of gamification devices, which form game mechanics that subsequently have quite good implications on the learning path.

### Gentle and gradual path

When approaching a new activity it is important to achieve **early satisfaction**, meaning that we should have the feeling of:

* Doing things right.
* Having potential to improve.
* Curiosity and interest.

{{< figure src="/images/post/hci/duolingo-gamification/easy-word.jpg" caption="An easy introductory question" class="right">}}

Obviously we won't learn a lot right at the beginning, but this phase is important to:

* Associate the new activity to a **positive feeling**.
* Form an **habit**.
* **Improve** ourselves.

Then, without we even noticing, we would gradually pass this *tutorial* phase and start to approach more difficult things.

If these phases are **seamless**, meaning that you cannot clearly distinguish the tutorial from the actual activity, the **positive reward** is even higher.

**Duolingo**—like some videogames—just puts you into the action and achieves early satisfaction by using a gentle and **gradual** path into the language to learn:

* Start with some basic and easy to pronounce words.
* Put them into short useful phrases.
* Repeat.
* **Gradually increase difficulty**.

That is the setup of the first lessons.

{{< figure src="/images/post/hci/duolingo-gamification/complex-phrase.jpg" caption="A more complex phrase" class="right">}}

Furthermore, graphically seeing lessons/topics in a **timeline**, helps to form two internal layers of learning metric:

* **Short term**: each lesson is a match.
* **Long term**: journey from zero to the end.

{{< figure src="/images/post/hci/duolingo-gamification/lessons-timeline.gif" caption="Duolingo lessons in a timeline" class="medium">}}

Another good element that helps to form the **habit** at the beginning, which is also an option that you set while on-boarding, is the **trainer**: you decide what is the **minimum amount of XP** (points) that you want to obtain each day; since the maximum that you can set is really low (50), it is very easy to get **early satisfaction**.

### Vanity metrics

The basic gamification elements of Duolingo are essentially all in this section; while using the app, the user collects:

* **Crowns**, after completing lessons.
* Rare crowns, after achieving the legendary status for a lesson.
* Unique **badges**, after completing monthly challenges.
* **Cards**, after unlocking some achievements.
* **Gems** and **lingots**: in-app pseudo-currencies that are earned in various ways and can be spent to:
	* Simplify your experience.
	* Get **access** to some challenges.
	* **Bet** to obtain even more gems/lingot.

The app also synthesizes your progress information into easily navigable **statistics** and **charts**.

### Challenges

This is where Duolingo starts to add some thin layering to its gamification.

#### Streaks

From the beginning the user is introduced to the concept of **streaks**: consecutive days where you complete at least one lesson. Again, we are still talking about the **learning habit**.

Why streaks work?

* It is satisfying to see your streak line increasing.
* It is terribly bad to see your streak line stopping and to start it over.

After each weeek of streak, Duolingo rewards you with a unique image to celebrate; you can also download the image and send it to your friends.

{{< figure src="/images/post/hci/duolingo-gamification/duolingo-streak.png" caption="A 30-day streak celebrative image" class="small">}}

#### Bets

This surprised me: sometimes the app asks you to **bet some in-game currency**. For instance it may ask you to bet 5 Lingots with the prospect of **doubling** it, if you reach a 7-day streak. Or you can bet 500 Gems and get 1000 if you reach a 30-day streak.

#### Monthly challenges

Each month, Duolingo poses a challenge to **earn 1000 XP within the month**. If you have an established learning habit it is quite **easy** to earn them and complete the challenge.

This challenge is ok for people that don't want a lot of competition to distract them, but appreciate the feeling of being in a **race** where there is no unique winner, but **anyone can win**.

At the end, you win a **unique badge**.

{{< figure src="/images/post/hci/duolingo-gamification/duolingo-monthly-challenge-win.png" caption="A monthly challenge win celebrative image" class="small">}}

#### Challenges among friends

You can add friends on Duolingo and **cooperate** to complete some challenges. For example, two friends can cooperate to jointly complete 30 perfect lessons (perfect == without errors) in 4 days.

When a user achieves something, **their friends get updated** on that and can cheer by pressing a button. Friends can also send **gifts** to each other, like some currency or *XP Potions*. Friends can also **encourage** each other.

{{< figure src="/images/post/hci/duolingo-gamification/duolingo-friends-updates.jpg" caption="Updates from fiends in Duolingo" class="right">}}

#### Championship and Leagues

For users that like some good competition, there is the possibility of setting the profile as public and to **compete with other users**.

Users are split into **leagues** (silver, gold, ...) based on their experience level, and each week there is a **championship**. Users in the same league don't compete with all other users of the same league, but according to a result of some matchmaking algorithm.

Users **earn XP** and go up/down the table. A the end of the week:

* The **1st** classified receives a special achievement.
* Users in the **top-3** receive another achievement.
* Users in the top-$k$ are **promoted** to the upper league.
* Users in the range $[k,q]$ remain in the league.
* All other users are **relegated** in the lower league.

For each league there is a fixed $k, q$.

### Lives

Lives are another classic example of basic gamification; within Duolingo, lives are regulated in this way:

* You have **max 5 lives**.
* One **error** costs you **1 life**.
* When you finish all lives, you cannot study new lessons.
* To gain a new life, you can either:
	* Review previous lessons.
	* Use Gems.
* Users with a Duolingo Super subscription have unlimited lives.

Lives are great because they make **failures cost something**; this is actually their gamification purpose:

1. One error.
2. One life less.
3. Angry.
4. Concentrated again.

When you finish your lives (or almost), the message is clear: you are not ready yet. How to solve it? How to improve? Simple: **review**.

Reviewing past and current topics kind of implements [spaced repetition](https://en.wikipedia.org/wiki/Spaced_repetition), which is a great technique to study things that vastly require mnemonic learning.

### Treasure chests

A treasure chest is a **timed device** that when active, **doubles every XP** that you earn.

When you complete a lesson (or just review something) in a couple of specific intervals of the day, you are rewarded with a treasure chest, that can be of type:

* Morning: 15 minutes.
* Evening: 15 minutes.
* XP Potion: 30 minutes.

An *XP Potion* is similar to a treasure chest but way more valuable, because it lasts 30 minutes. XP Potions are randomly unlocked by just completing lessons.

{{< figure src="/images/post/hci/duolingo-gamification/treasures.jpg" caption="Obtaining an XP potion in Duolingo" class="small">}}

Treasure chests are useful to:

* Establish a **regular learning habit**.
* Maintain the habit.

### Repairing lessons

Still speaking about **spaced repetition**, Duolingo also implements it with the metaphor of a lesson that is about to **break**.

{{< figure src="/images/post/hci/duolingo-gamification/lesson-breaking.jpg" caption="A lesson needing repair in Duolingo" class="small">}}

When you see some cracks on your shiny lesson circle, it means that you must **repair** it. It happens every few days.

If you have a substantial number of completed lessons, this **forced maintenance** may feel like an annoyance: that is why you are rewarded a little more and there is no life subtraction.

### Legendary status

Each lesson has **5 levels** of **increasing difficulty**. On top of the fifth level there is the so called **Legendary level**.

To win it, you must complete a certain number of rounds, without help, with max 3 errors allowed. Each round gives you 40XP.

Essentially, there is **more risk and more reward**.

This is where the competitive players reveals themselves: do you need a stunt in the league table? Just complete a couple of rounds for the legendary status while having a treasure chest:

$2 * 2 * 40\ XP = 160\ XP$

Here the gamification effect is more derived from a **personal satisfaction** point of view: when you reach legendary, you feel that you own that lesson, which is a **positive reinforcement** that is useful to gain more motivation.

## Conclusion

In this post we have seen multiple examples of how Duolingo uses gamification to motivate the user, and how they combine and layer different gamification devices.

This heavy gamification usage ultimately produces positive results, but also some negative ones.

### The good

* Gentle introduction to the language.
* Early satisfaction.
* Small bits of information.
* You decide your rhythm.
* Seamless transition from easy to hard.
* Feeling of getting better.
* Metrics motivate you to do more.
* Streaks produce habits.
* By using the same words and topics:
	- More focus on the language patterns.
	- Less focus on single words.

### The bad

* After a while, repairing lessons feels like a chore.
* If you are competitive, it is easy to degenerate.
* It is demotivating when a correct answer is wrongly not accepted.
* If you miss one day because of some important reason and lose your streak, the negative effect will be too high, probably demotivating.