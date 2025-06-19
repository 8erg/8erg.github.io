+++
title = 'Replay Attack on a Ford Fusion'
date = 2024-10-16
draft = false
+++




# Table of Contents

- [1. Introduction](#1-introduction)
- [2. Walkthrough](#2-walkthrough)
  - [2.1 Finding the key frequency](#21-finding-the-key-frequency)
  - [2.2 Validating the key frequency](#22-validating-the-key-frequency)
  - [2.3 Analyzing the recorded signal](#23-analyzing-the-recorded-signal)
  - [2.4 The fateful day](#24-the-fateful-day)
  - [2.5 More In-Depth](#25-more-in-depth)
- [3. Conclusion](#3-conclusion)

---

## 1. Introduction
Last October I was able to perform my first key signal replay attack on a car with a friend. I was dealing with RF and stuff and wanted to try this attack, but at the time i didn't have an `hackrf` (yes, i'm a brokie...), so I asked my friend and we decided to do it together. Important to note, that this was performed for research and learning purposes and I'm not promoting illegal activities. All the test we're perform with authorization from the owner of the car.


>[!warning] Important
>Important to note, that I did not take any pictures, only a video showing the unlocking of the car. Sadly, I don't really have access to the car anymore to take pictures of the processes, but I will be detailing the thought processes and the hurdles we faced throughout this experience

---

## 2. Walkthrough

#### 2.1 Finding the key frequency
So first thing first we had to figure out what frequency the key was operation on, since it was hard to read it on the key fob we decided to search for it on the internet. After a couple search we found that the key was operating at `315MHz` frequency. during my research, I also stumbled upon a database for all the `FCC ID` which track and verify all electronic devices.

>[!tip] Link
>Here's the link, if you ever want to check one of your electronic devices : `https://fccid.io/`

#### 2.2 Validating the key frequency
I personally had a raspberry pi which was running `dragonos` which contain a bunch of `RF` tools. To validate the frequency i started `gqrx` and put myself on the required frequency and then pressed the key fob. As a first time experiencing this, it was pretty exhilarating, even though it's nothing. One thing to note I did this prior to meeting with my friend who had the `hackrf` and I used and `rtl-sdr` which cost roughly around `30$` you can find it on AliExpress, I'll put a link down below.

#### 2.3 Analyzing the recorded signal
Before going to meet my friend, I tried analyzing the signal inside `Universal Radio Hacker`
which is fantastic tool for `Reverse engineering` radio frequencies. By the way, I would like to say that reverse engineering is the best skill to have, hands down. Anyway, back to the main subject, so I tried reversing it, but i couldn't make anything out of it and to be honest, i didn't think it was necessary to reverse the signal to be able to perform a replay attack, but for the sake of learning purposes i still tried (my skill level in RF aren't that high, so I quickly gave up).

#### 2.4 The fateful day
So here I am, in the black ford fusion, ready to perform the attack of the year with my most formidable acolyte...in a parking lot, broad day light....😂

So we meetup, we parked the car and we get started. But first and foremost, I think to myself we should get the windows down, because there could be some sort of security trigger that could lock us out. Thank god we did because on our first try we triggered the safe lock system.

#### 2.5 More In-Depth
We plugged the `hackRF` and started `Universal Radio Hacker` and then we recorded the signal through `URH`. We tried to replay it, the first signal went out and nothing happened. We send it a second time and nothing happened. We send it again and then....boom! The safe lock system got triggered. the alarm car started blasting, everyone started looking and we started panicking. Lucky for us we kept the windows open. So i went in and put the key, start the car and the alarm stopped.

We tried the same steps multiple times and landed with the same results. While experimenting, we also took time to analyze the signal inside `URH`, we were able to deduce that the signal generally had an header and a body (of course this were all hypothesis) and that the header possibly contained some sort of counter. Every time, the signal that we replayed was more than two time behind the counter of the car, the safe lock system would get triggered.

So I thought to myself what if we were to record a signal outside of the range of the car, which would be considered a fresh signal. So here we are, my friend had his laptop with the `hackRF` and we were just walking around with it, innocently...😂Now that we were outside of the range, we registered the signal (btw the range of the car was big).

Walking back to the car, with uncertainty, with doubts, is this going to work? Are we on the verge of becoming certified car hackers...?(why this guy is trying to sound overly dramatic..😂). Anyway we come back to the car, replay the signal and the car unlocked itself. I could just feel the dopamine rush, honestly it was great feeling!

---

### 3. Conclusion

It was a great experience, I really waited too a long time to write this blog post. I don't think this experiment is really practical in a real word scenario. But I think that if a single person was targeted, it could be more feasible. You would have to target a person and be close to that person when he unlocks his car and then reverse engineering it with `Universale Radio Hacker` to understand the protocol used to be able to perform something with it, but it's highly unlikely to be able to perform this, without a good preparation. Do you had to hack a car for an engagement or just for fun?