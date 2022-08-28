---
author: Davide Quaranta
title: Creating vocal interfaces is difficult
date: 2021-11-08
categories: [hci]
tags: [opinion]
description: Why is it so complex to create a good voice interface? There are many factors that affect the quality and usability of the final product. There are many variables involved, and the technology choices are decisive. Let's see why.
toc: true
---

What is a voice interface? Or Vocal UI to put it more coolly?

It's a type of interface that the user doesn't see, but listens to and interacts with by voice. Examples might be Cortana, Alexa, Siri, Google Coso, or anything that is voice-activated, and responds to you with lots of voice.

As interesting as the idea of having a virtual assistant that talks and listens may be, building such an interface is not at all easy. Or rather, it is (relatively) easy if your goal is easy, but it gets dizzyingly complicated as the complexity of the *tasks* you want to accomplish increases.

This article contains personal reflections and opinions that I developed while creating the final project for the Human Computer Interaction (HCI) course at the master's program I am taking. 

Together with four colleagues, we decided to create a voice interface to maneuver some [CRM](https://it.wikipedia.org/wiki/Customer_relationship_management); specifically, we took a lot of inspiration from [Monica](https://www.monicahq.com/), a person-oriented CRM instead of a business-oriented CRM. We developed our interface as a *skill* for [Mycroft,](https://mycroft.ai/) an open-source voice assistant.

Very simply, a CRM is a program for managing relationships with people, such as noting the last time we spoke with them, how, when, why, etc. Imagine an address book, but with muscle.

Already from this description it is easy to get a first idea of the possible difficulties in creating such a thing. In fact, the points I write in the article come from experiences and experiments we made during the development of the project.

Let's start.

## Voice recognition.

The first obvious thing an interface has to do to understand what you said, is to take the audio recording of your pretty little voice, and feed it to a black box that spits out the transcript (**speech to text**). All this is called **speech recognition**.

What are the problems in doing something like this? What can go wrong?

### Quality of audio input.

Here are a few scenarios:

- **Recording volume is too low,** and the recognition engine fails to do its job properly; it may, for example, skip whole words.
- **Echo cancellation**: Depending on the device you run the voice assistant software on, there may or may not be echo cancellation. In any case, poor or absent echo cancellation can compromise quality, for example, if the room acoustics are bad. Watch out, because excessive echo cancellation can also give problems.
- **Noise cancellation**: as with the previous point, if the environment is very noisy and the threshold of the *gate* is too low, ambient noises could be interpreted as words. Conversely, too high a threshold may have the effect of mistaking correctly spoken words for noise.
- **voice timbres**: Depending on the device hardware and the speech-to-text model, some voice timbres may be more or less recognizable. For example, during the testing of our examination project we noticed that the Mycroft STT engine we were relying on had problems recognizing fine/high voices.

These points are very dependent on the hardware you are using (device, microphone) and the software running on it (OS, audio drivers, filters, voice assistant).

A fun thing I did to see if I could solve the problems of recognizing "fine" voices with some filters was to put an [octaver](https://it.wikipedia.org/wiki/Octaver) before Mycroft and... surprise, fine voices were being recognized.

### Dialects, different languages, unknown words.

In the ideal world, everyone speaks one linuga correctly, under favorable environmental conditions, has impeccable diction, doesn't say things like *"uhmmmm, that is is typeoo, eeeeeee "*, there are no dialects, no neologisms, and no one invents new terms.

In the real world, we all speak in our own way, even more than one language, we use words of different languages in the same sentence, we talk while the dog barks, we make sounds while we think, we make up words, and sometimes we replace names of objects with *"thing "*.

We humans are good at disambiguating all these things; an algorithm might be more or less capable, but in general we say no.

## Intent parsing

The text extracted from the user's **utterance** (what he said), must be understood in some way: the system must derive meaning from it.

The concept of meaning can be reduced to the concept of **intent,** that is, what the system believes is the user's intention.

To give an example, let us imagine that our voice interface manages a small smart home system, where the possible intents are:

- Turn on light number X.
- Turn off light number X.
- Increase the brightness of light number X.
- Decrease the brightness of light number X.

When the user speaks, the system has to figure out what to do among the things in this list; to do this, it needs some rules. This process is called **intent parsing**, and it consists in short of finding a match between the user's utterance and an intent defined by the person who created the skill.

Intent parsers can work differently: for example [Padatious](https://github.com/MycroftAI/padatious) is based on neural networks, while [Adapt](https://github.com/MycroftAI/adapt) is based on keyword recognition.

Depending on the complexity of the task and the characteristics of the intent parser, things can get complicated, especially if the system also has to extract entities. For example, our intent definition for Padatious of our silly little smart home system might look like this.

I don't feel like writing For brevity's sake, I omit the complementary tasks of shutdown and decrease.

The syntax is easy:

- `(a|b|c)` denotes optionality of strings {a, b, c}.
- `(|a|b|c)` adds the empty string to optionality.
- `{Entity}` is a placeholder for an entity named `Entity`.

Although these are very simple tasks, they are still suitable for reasoning about what can go wrong. What happens if the user says something like:

> Ehmm, so, can you please turn up the brightness of the lighteeehin the kitchen? Thank you.

You only have to do some usability testing to see that users tend to shoot utterances like that. Obviously the less experience the user has with voice assistants, the more they will tend to construct "polite," unstructured, complex, longer-than-necessary utterances.

If the intent parser is good at filtering out all possible stopwords and other undesirable words (*thanks, please, um, then,* etc), the interaction will be quite pleasant, and we should not have to work hard to write good intent definitions and manage them. If we consider Padatious, we will find that it is not good at it at all. In particular, during our project we realized that:

- Intent definitions with and without entities work poorly together.
- For some reason, the parser works poorly when there are many optionalities.
- On some occasions, the content of the extracted entities is wrong. [[1](https://github.com/MycroftAI/padatious/issues/4)][[2](https://github.com/MycroftAI/padatious/issues/25)]

Having found these problems, we might well consider moving to another intent parser, such as Adapt. Migrating is simple: just rewrite all intent definitions as RegEx, define files containing keywords, and declaratively annotate how we expect the user to use the combination of keywords and intent definitions.

I'll give an example so you understand better:

To handle the first task we might have:

Where `LightName` is mandatory, and corresponds to the name of the entity to be extracted, and `PowerOnKeyword` is equally mandatory and is the name of a file containing a list of keywords that we expect the user to say to turn on a light, such as {turn on, turn on}.

This intent parser looks very good and seems very flexible, in fact it is. Too bad it opens up a whole new world of problems to handle:

- Writing intent definitions that are too generic can cause conflicts (which we will see in a moment).
- `.*` is very convenient, but if abused it can cause conflicts.
- Multiple RegEx can match the same utterance; it is a problem when we expect to find an entity but the RegEx without entity is matched.
- In some cases, especially if the RegExs are complex or very diverse, Adapt fails to match.

In our project, the last point gave us a lot of work; in particular, in order to land the user on the correct task even in the absence of a complete match, we had to remove the evaluation of RegExs from Adapt:

- With Adapt we only checked for the presence of certain keywords.
- RegEx evaluation was handled by us, to extract the right entities.

We basically rewrote our own code to do matching on RegExs, because the one used by Adapt was unreliable and failed.

## Conflicting intents.

The probability of having conflicts increases as the number of tasks you handle and the number of skills installed by the user increase.

A conflict occurs when an utterance is reducible to more than one intent. The more generic the intent definitions are, the easier it is to have conflicts.

If one of your tasks conflicts with that of another skill, good luck.

## Error correction.

When we use graphical interfaces, it is (relatively) easy to handle errors. It is a fairly common practice to use color code, icons, and text to highlight errors (think of a form to fill out, for example).

Handling errors by voice alone, without visual reference, can be frustrating, annoying, slow, and confusing. The worst case is when the user does not understand where the error is; therefore, we must try to always offer **contextual help**.

What happens if we ask the user to give a name to a new contact in the address book, and the user answers "the last name is Di Paolo"? Clearly we have to be good at filtering out all that filth that clumsy users tend to pronounce.

But what happens if not the user says something really weird, that gets past our filters?

What happens if the user says "this person's last name is Di Paolo."

Suppose we have a filter that:

- Deletes {name} as it is a word that we consider undesirable.
- Deletes {the, of, is} as it is stopwords.

We are left with "this person Paul." Bad thing: we lost a piece of the last name. How should we treat this phrase? Do we take it at face value? Do we need a better filter? 

During the project we had to handle these situations, which in our case were quite frequent because unlike the little system for turning lights on and off, in our case the user had to **generate content**.

We used two approaches depending on when these situations were happening:

- In certain cases, such as when the user had to provide the first and last name of a new contact, if stopwords remained after removing obvious unwanted words, the interface would ask for confirmation.
- In less critical cases, simply the interface would accept the utterance and repeat it back to the user, so that the user would post evaluate for himself whether to repeat or not.
- In cases where the complete integrity of the utterance was not important (e.g., on a search), the interface would use the fully filtered output.

None of these approaches is absolutely correct: it depends on the context.

## Flow control.

This point is closely related to the previous one. If your task is complex (consisting of multiple steps, a dialog), it is appropriate to give the user the ability to manage the flow of the task, such as repeating something, going back, skipping unimportant steps, or aborting execution.

Providing flow control is simple, but it opens up a world of complications: suppose we set key words for flow control: {forward, backward, skip, cancel}, plus some variant, synonym, or words with similar meaning (e.g., in our project in some cases we were evaluating expressions such as "I don't remember" as a way to skip a question).

At this point managing these words gets a bit complicated: what happens if a contact's last name includes "skip"?

In our project, the interface asked for confirmation when the situation was ambiguous; when it did not ask for it, the user could eventually correct his error because in each case he received feedback on the input "understood" by the interface, and could correct the error by repeating the task piece.

Right, wrong, elegant, inconvenient? Perhaps it matters little: usability test results tell us that users were able to finish tasks with fewer errors and with more ease.

## Context management.

When we talk to someone, a **context,** i.e., a set of information related to the conversation, describing its state, is formed. The context is continuously enriched as the conversation continues.

In a speech interface with complex tasks, the same phenomenon occurs, or rather, must occur; it is we developers who must reproduce it.

In particular, we have to make sure that all the data that the user gives us as we go along is added to this context, so as to provide a kind of **short term memory** to our interface, or even long term if we are particularly ambitious.

Managing context is by no means trivial, but Mycroft helps us through a pattern called [Conversational Context](https://mycroft-ai.gitbook.io/docs/skill-development/user-interaction/conversational-context), in which each task step corresponds to a function with a specific annotation.


## Conclusion.

Creating speech interfaces is difficult. There are many factors that affect the quality and usability of the final product:

- Everything about the hardware of the fisisco device on which the system runs, including the microphone.
- The operating system and audio drivers installed.
- Any upstream filters.
- The speech-to-text engine.
- The intent parser and the quality of intent definitions.
- We have not mentioned this because it is outside the scope of this article, but the text-to-text engine also affects quality: think of voices that are unnatural or for some reason are not pleasant.

In short, there are many variables at play, and technology choices are key.

Great, if you liked the article let me know in some way. Then if you know someone to whom it might be useful, consider sending it to them :)