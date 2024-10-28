# Note Types to Avoid Pattern Matching

One of the big issues that Anki users face is memorizing what the answer *looks* like rather than the actual information, which is sometimes called "pattern matching". This can lead to situations where someone can "recall" the answer in Anki, but not in real life. The new note types that I wrote about in this article aim to solve this problem as well as allow you to memorize the same amount of information while making fewer cards. 

[https://ankiweb.net/shared/info/171015247](https://ankiweb.net/shared/info/171015247). The deck has examples of 5 new note types: Match Pairs, Randomized Cloze, Randomized Basic, Randomized Basic with Multiple Answers, and Click Words. Once you download it, you'll be able to make cards based on these note types on your own, **no add-ons needed**. Huge thanks to [Vilhelm Ian](https://github.com/Vilhelm-Ian) (aka Yoko in the [Anki Discord server](https://discord.gg/qjzcRTx), aka [AnkiQueen](https://forums.ankiweb.net/u/ankiqueen/summary) on the forum) for making these note types!

They work on PC and on AnkiDroid, but haven't been tested on AnkiMobile.

Table of contents:
- [Match Pairs](#match-pairs)
- [Randomized Cloze](#randomized-cloze)
- [Randomized Basic](#randomized-basic)
- [Randomized Basic with Multiple Answers](#randomized-basic-with-multiple-answers)
- [Click Words](#click-words)


## Match Pairs

Have you ever had cards like this? There are 2 pieces of knowledge, and you can't remember which is which, so you make a Cloze.

![1](https://github.com/user-attachments/assets/def904c9-b78b-437f-ac66-d8f0807f155f)

But there is a problem: you may end up just memorizing "thingy 1 is the top one, thingy 2 is the bottom one". In order to avoid that, you could make two notes with the order switched.

![2](https://github.com/user-attachments/assets/8ef6dd0a-6417-4106-a422-10048bd67af6)

However, this is inefficient - now you have two notes even though theoretically you only need one. If only there was a way to put them into the same note and randomize the order...

Well, with Match Pairs there is!

![Match Pairs (Sweden and Switzerland)](https://github.com/user-attachments/assets/b43b20f1-f267-49eb-ae97-a65156ffc159)

And if you think that this is too easy and therefore would make active recall ineffective, you can make your life harder by adding a wrong answer.

![Match pairs (one wrong)](https://github.com/user-attachments/assets/0311774a-abc0-4e9d-9a24-edb9a93bddc9)

Here you have 2 countries and 3 capitals, so you need to think harder.<br />
Make sure that the extra answer is wrong, but not *obviously* wrong. In this example, I won't benefit from adding Jakarta to the second list, since it's obviously wrong. Which is why I added Amsterdam - Amsterdam makes me pause and think, Jakarta doesn't.

Still not hard enough? You can add 2 wrong answers. The number of wrong answers displayed is at most equal to the number of correct answers. The card below will never show "Poopville", because there are 2 correct answers, which means that there can only be 0, 1 or 2 incorrect answers.

![image](https://github.com/user-attachments/assets/c1a4eb0e-d218-4562-8a0d-f5d9e4a77d20)

Btw, you don't necessarily have to drag answers - you can click on them. When you click on an answer, it is put in the topmost vacant answer box.

`|` is the separator that you should put between items, **this is all you have to remember to create these cards**. Don't worry about leading/trailing spaces, they are stripped away automatically: `Answer1 | Answer2` will produce the same result as `Answer1|Answer2`.

In all examples above, I used two pairs, but you can add more. However, stuffing too much information into a single card is a bad practice. I recommend having 2-3 pairs, *maaaaaaaaaaaybe* 4, but not more.

Match Pairs also supports images.

![Match pairs (image)](https://github.com/user-attachments/assets/e4239938-a926-43dd-ad61-4d53c331247c)

And audio (if you can't hear it in the example below, make sure to click the speaker symbol ![image](https://github.com/user-attachments/assets/9d3d1efb-8669-484d-91bb-e1c7a91b7b30)).

https://github.com/user-attachments/assets/6f65fb06-322b-4745-8ee9-101b126e2df5

Of course, how useful this note type is for you depends on how often you encounter what I call "negative interference", where card A makes it harder to remember card B, and card B makes it harder to remember card A. Personally, I've been able to replace dozens of unnecessary clozes with this note type, and I think it would be cool if this note type would become built-in in the future.


## Randomized Cloze

This is another note type that aims to solve the pattern matching problem.

![Randomized Cloze](https://github.com/user-attachments/assets/4cfd6931-477e-4810-9465-33078f1ff8be)

To save some time and effort, you can ask ChatGPT, Claude or Gemini to rephrase the sentence and generate 2-3 sentences with the same meaning, although I recommend taking the time to write sentences yourself.

One thing that you should keep in mind: the numbers in curly brackets have to be the same for each item, otherwise you'll end up making multiple cards instead of one card. It doesn't mean that the number always has to be 1, you absolutely can have multiple cloze selections per item. Like this: `Just some {{c1::random}} {{c2::text}}| Also just some {{c1::random}} {{c2::text}} | And this is some {{c1::random}} {{c2::text}}, too`.

![Cloze edit](https://github.com/user-attachments/assets/764d993c-0782-4816-90bf-7d0f4f13058f)

The `|` separator is the same.


## Randomized Basic

It's exactly what it sounds like. And the separator is the same.

![Randomized Basic](https://github.com/user-attachments/assets/47023e01-bbf3-4011-a47f-3a7d53a8a240)

![image](https://github.com/user-attachments/assets/34b927b5-7342-4c2f-83a7-9d881002fec7)

Keep in mind that this isn't Match Pairs, the back can only have **one** item. The `|` separator won't work in the "Back" field.


## Randomized Basic with Multiple Answers

This is just 2/3/n notes in one. You may be wondering, "Why not just *actually* make several notes?". For the most part that's true, but there is (at least) one situation where this is useful: practicing math concepts.

![RandomBasic with MA](https://github.com/user-attachments/assets/57269c25-e82f-4e2c-a733-f631871836d4)

![image](https://github.com/user-attachments/assets/902bc3d9-5411-4ca5-b39d-16ff225e1a3e)

You could make 3 separate notes, but then you would have 3 notes (and cards) for the same concept, which is less efficient.

Here's a little diagram to help you understand the difference between this and Randomized Basic.

![RBasic vs RBasic with MA](https://github.com/user-attachments/assets/ef9ce821-61aa-44af-b534-d5c453ac0dc7)


## Click Words

![Click Words](https://github.com/user-attachments/assets/02d7bc6f-2a1d-4b7e-818d-9b4f30631caa)

![image](https://github.com/user-attachments/assets/80454f13-30f3-4733-b23b-6a9c9d3ccf37)

"Title" is an extra field, you can leave it empty, if you want.

I don't really like this note type. It's like Cloze, but with multiple answers. I believe this isn't beneficial since it makes recall much easier than cloze, which isn't good for strengthening memories, and the only "advantage" is that it looks fancy. Just use Cloze, or even better - Randomized Cloze.

All note types will notify you if the creator has released a new version on [AnkiWeb](https://ankiweb.net/shared/info/171015247):

![image](https://github.com/user-attachments/assets/464b7ae3-af82-40e4-a1df-5b1549348f65)

---

P.S. When you download the deck, there will be this card:

![image](https://github.com/user-attachments/assets/ff46142b-776b-479a-bf9a-884e76761ef3)

As it says, don't delete it. It is necessary for some stuff related to playing audio in Match Pairs. This card is suspended by default, to avoid confusing people.

# If you find any bugs or if you have any feature requests, here: [https://github.com/Vilhelm-Ian/Interactive_And_Randomize_Anki_Note_Types/issues/new](https://github.com/Vilhelm-Ian/Interactive_And_Randomize_Anki_Note_Types/issues/new)


___
### [←Return to homepage](https://expertium.github.io/)
