# Understanding retention in FSRS

TLDR: desired retention is "I will recall this % of cards **WHEN THEY ARE DUE**". Average retention is "I will recall this % of **ALL** my cards **TODAY**".

In Anki, there are 3 things with "retention" in their names: desired retention, true retention, and average predicted retention.

Desired retention is what you want. It's your way of telling the algorithm "I want to successfully recall x% of cards **when they are due**" (that's an important nuance).

True retention (download the Helper add-on and Shift + Left Mouse Click on Stats) is measured from your review history. Ideally, it should be close to the desired retention. If it deviates from desired retention a lot, there isn't much you can do about it.

Basically, desired retention is what you want, and true retention is what you get. The closer they are, the better.

Average predicted retention is very different, and unless you took a loooooooong break from Anki, it's higher than the other two. If your desired retention is x%, that means that cards will become due once their probability of recall falls below that threshold. But what about other cards? Cards that aren't due today have a >x% probability of being recalled today. They haven't fallen below the threshold. So suppose you have 10,000 cards, and 100 of them are due today. That means you have 9,900 cards with a probability of recall above the threshold. Most of your cards will be above the threshold most of the time, assuming no breaks from Anki.

Average predicted retention is the average probability of recalling **any** card from your deck/collection **today**. It is FSRS's best attempt to estimate how much stuff you actually know. It basically says "Today you should be able to recall this % of all your cards!". Maybe it shouldn't be called "retention", but me and LMSherlock have bashed our heads against a wall many times while trying to come up with a naming convention that isn't utterly confusing and gave up.

I'm sure that to many, this still sounds like I'm just juggling words around, so here's an image.

![Desired retention](https://github.com/user-attachments/assets/44508803-458f-44b8-ae0d-c8525dc7148c)

On the x axis, we have time in days. On the y axis, we have the probability of recalling a card, which decreases as time passes. If the probability is x%, it means that given an infinitely large number of cards, you would successfully recall x% of those cards, and thus your retention would be x%.

Average retention is the average value of the forgetting curve function over an interval from 0 to whatever corresponds to desired retention, in this case, 1 day for desired retention=90% (memory stability=1 day in this example). So in this case, it's the average value of the forgetting curve on the [0 days, 1 day] interval. And no, it's not just (90%+100%)/2=95%, even if it looks that way at first glance. Calculating the average value requires integrating the forgetting curve function.

If I change the value of desired retention, the average retention will, of course, also change. You will see how exactly a little later.

Alright, so that's the theory. But what does FSRS actually do in practice in order to show you this number?

![FSRS average retention](https://github.com/user-attachments/assets/cf4f5cb0-d049-400e-97b4-dc5ba14b0df1)

It just does things the hard way - it goes over every single card in your deck/collection, records the current probability of recalling that card, then calculates a simple arithmetic average of those values. If FSRS is accurate, this number will be accurate as well. If FSRS is inaccurate, this number will also be inaccurate.

Finally, here's the an important graph:

![Average retention vs Desired retention](https://github.com/user-attachments/assets/63126764-0564-4f9e-90ed-a26a734e1193)

This graph shows you how average retention depends on desired retention, in theory. For example, if your desired retention is 90%, you will remember about 94.7% of all your cards. Again, since FSRS may or may not be accurate for you, if you set your desired retention to 90%, your average predicted retention in Stats isn't necessarily going to be exactly 94.7%.

Again, just to make it clear in case you are lost: desired retention is "I will recall this % of cards **WHEN THEY ARE DUE**". Average retention is "I will recall this % of **ALL** my cards **TODAY**".


___
### [←Return to homepage](https://expertium.github.io/)
