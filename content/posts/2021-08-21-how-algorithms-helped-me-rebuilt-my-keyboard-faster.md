---
title: "How Algorithms Helped Me Rebuilt My Keyboard Faster"
date: 2021-08-21T15:09:39-03:00
categories:
  - Algorithms
tags:
  - Algorithms
  - HashTable
  - Technology
featured_image: /images/posts/2021-08-21-how-algorithms-helped-me-rebuilt-my-keyboard-faster/1.JPG
---

I decided to clean up my keyboard this weekend and it was about time. After more than a year of use, it was shameful and disgustingly filthy.

I started follwing the drill. I watched some Youtube videos to get the idea over what I was about to do and to reduce the chances of accidentally terminate for good my Redragon.

Time to get my hands dirty. With my plastic pot, paint brush and key cap remover in hands, I dig into it, starting by removing my keyboard 105 keycaps. Man that's some piece of job.

With a key cap naked keyboard, fase 2 started: remove all dirty from the keyboard with the paint brush. And man, until this time I was unable to realized how much of everthing a mechanical keyboard is able to retain. It felt like my whole apartment was beneath theses plastic caps.

After cleaning all dust, crumbs and dog fur out of the Redragon, time to phase 3: dry up all wet keycaps. That was, definitively the most boring part of all. To ensure they were not wet and wouldn't damage the eletronics of the keyboard, I dried up, one by one, the 105 caps.

Boring part finished, time to the pleasant and final part: put all caps back on the keyboard shell. However, the beauty quickly became the beast. Since I assembled all the keycaps on the table randomly and started picked them up randomly too, I was taking forever to fill up a single row of the keyboard. I thouth man, I need some system. Something to make this process streamlined, less random and caothic. I need an algorithm.

By definition, an algorithm is simply "a set of instructions that must be followed in a fixed order". So, I came up with a simple one:

> Put the caps back from bottom to top, from left to right. Look which cap goes on that specific gap, find the cap on the pile of caps, put it on and move to the next gap.

![Keyboard without some keycaps](/images/posts/2021-08-21-how-algorithms-helped-me-rebuilt-my-keyboard-faster/2.JPG)

There were just one problem, specifically with **find the cap on the pile of caps**. Since the keycaps were randomly disposed on the table, each time I needed to find an specific cap, I needed to seach it among all the other dozens of plastic parts, performing a full scan seach with my brain. It was hellish, insane, and slow hard work.

When I suddenly had a click: Dude, this is a search problem! Each time I need a new cap, I need to go over all the keycaps randomly to find it and it is taking forever. To fix it, I just need a better form to organize the caps for searching. A better data structure. And for me, the clear choice was a **Hash Table**.

A **Hash Table** is data structure where you a take a whole buch of items and store them grouped according to some sort of order. This way, if you have 100 items and 10 groups, instead of searching over the 100's to find an item, you find out which group this item belongs to, and narrow down dramatically the number of items we need to look up.

So that's exactly what I did. I divided the keycaps into 4 groups: **letters, numbers, function keys (F1, F2, and so on) and special keys (Enter, PgDown, Space bar, etc)** and ran the same algorithm with a slight change:

> Put the caps back from bottom to top, from left to right. Look which cap goes on that specific gap, **find to which group this cap belongs**, find the cap on the **group** of caps, put it on and move to the next gap.

![Keyboard and keycaps separated by groups](/images/posts/2021-08-21-how-algorithms-helped-me-rebuilt-my-keyboard-faster/3.JPG)

Man it was magical! I just grouped the keycaps into simple categories and I finished the whole job within a couple of minutes. I could've done it faster, but I accidentally misplaced the `R` key and had to pop out half of the keyboard row.

Dude, it was awesome to realize how common software engineering solutions are really impactful in the way we organize and search information. I'm really glad that women and men before me put in the hard work to create these great ideas so I don't need to cave them from scratch while putting 105 keycaps back on my keyboard!

---

If you wanna know more about **Hash Tables** and how they work in real software, this article by [@dorin131](https://github.com/dorin131/) might be a good place to start: https://dev.to/dorin/data-structures-in-go-hash-table-55lg.

---

> “I'd far rather be happy than right any day.”
> ― Douglas Adams, The Hitchhiker's Guide to the Galaxy
