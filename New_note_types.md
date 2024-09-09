# New note types

[https://ankiweb.net/shared/info/171015247](https://ankiweb.net/shared/info/171015247). The deck has examples of 4 new note types: Match Pairs, Randomized Cloze, Randomized Basic and Click Words. Once you download it, you'll be able to make cards based on these note types on your own, no add-ons needed.

- [Match Pairs](#match-pairs)
- [Randomized Cloze](#randomized-cloze)
- [Randomized Basic](#randomized-basic)
- [Click Words](#click-words)

## Match Pairs

Have you ever had cards like this? There are 2 pieces of knowledge, and you can't remember which is which, so you make a Cloze.

![1](https://github.com/user-attachments/assets/def904c9-b78b-437f-ac66-d8f0807f155f)

But there is a problem: you may end up just memorizing "thingy 1 is the top one, thingy 2 is the bottom one". In order to avoid that, you could make two notes with the order switched.

![2](https://github.com/user-attachments/assets/8ef6dd0a-6417-4106-a422-10048bd67af6)

However, this is inefficient - now you have 2 notes for the same thing. If only there was a way to put them into the same note and randomize order...

Well, with Match Pairs there is!

![2 capitals](https://github.com/user-attachments/assets/905f5a88-33f4-419a-a662-e1906c835385)

And it's not the only advantage of this note type. It also supports images.

![Images](https://github.com/user-attachments/assets/2b648768-12d4-4755-a036-d0ba7681c416)

And audio (if you can't hear it in the example below, make sure to click the speaker symbol ![image](https://github.com/user-attachments/assets/9d3d1efb-8669-484d-91bb-e1c7a91b7b30)).

https://github.com/user-attachments/assets/6f65fb06-322b-4745-8ee9-101b126e2df5

And if you think that this is too easy and therefore would make active recall ineffective, you can make your life harder by adding one or two fake answers.

![3 capitals](https://github.com/user-attachments/assets/4133a134-4501-4a7b-a17b-a562d0ec3228)

Here you have 2 countries but 3 capitals, so you can't answer correctly if you only know the capital of one country but not the capital of the other one. <br />
Make sure that the extra answer is wrong, but not *obviously* wrong. In this example, I won't benefit from adding Jakarta to the second list, since it's obviously wrong. Which is why I added Amsterdam. I don't have to think very hard to remember that Jakarta is neither the capital of Sweden nor Switzerland, whereas Amsterdam makes me pause and think.

Btw, you don't necessarily have to drag answers - you can click on them. When you click on an answer, it is put in the *top* box.

And here's what the fields look like:

![image](https://github.com/user-attachments/assets/87a2b1b6-231b-40c2-8934-b0f9977b1cd8)

`|` is the separator that you should put between items. Don't worry about leading/trailing spaces, they are stripped away automatically: `Answer1 | Answer2` will produce the same result as `Answer1|Answer2`.


When you download the deck, there will be this card:

![image](https://github.com/user-attachments/assets/ff46142b-776b-479a-bf9a-884e76761ef3)

As it says, don't delete it. I won't go into the details regarding this, but basically Match Pairs doesn't play audio automatically. It's not a bug, it's a feature.

In all examples above, I used two pairs, but you can add more. However, stuffing too much information into a single card is a bad practice.

Of course, how useful this note type is for you depends on how often you encounter what I call "negative interference", where card A makes it harder to remember card B, and card B makes it harder to remember card A. Personally, I really wish that this note type would become a built-in type. Personally, I've been able to replace dozens of unnecessary clozes with this note type.

## Randomized Cloze

This is another note type that aims to solve the "memorizing the shape of the question rather than the content" problem.

![Randomized Cloze](https://github.com/user-attachments/assets/75c665cc-470a-4930-b527-ef1e586ab04b)

To save some time and effort, you can ask ChatGPT, Claude or Gemini to rephrase the sentence and generate 2-3 sentences with the same meaning, although I recommend taking the time to write sentences yourself.

One thing that you should keep in mind: the numbers in curly brackets have to be the same for each item, otherwise you'll end up making multiple cards instead of one card. It doesn't mean that the number always has to be 1, you absolutely can have multiple cloze selections per item. Like this: `Just some {{c1::random}} {{c2::text}}| Also just some {{c1::random}} {{c2::text}} | And this is some {{c1::random}} {{c2::text}}, too`.

The separator is the same as for Match Pairs.

![Cloze edit](https://github.com/user-attachments/assets/5881adc5-7150-4f3a-8efa-90517425eb7a)

## Randomized Basic

It's exactly what it sounds like. And the separator is the same.

![Randomized Basic](https://github.com/user-attachments/assets/7a4e54a4-5646-4f3c-8a25-7f3dfc9f3f54)

![image](https://github.com/user-attachments/assets/5df3b54d-3e07-4227-ba98-c420054674ae)

Keep in mind that this isn't Match Pairs, the back can only have **one** item. The `|` separator won't work.

Here's a little diagram to help you.

![Match Pairs vs Randomized Basic](https://github.com/user-attachments/assets/3926eee6-4146-403c-be11-e5e96775f151)


## Click Words

![Click Words](https://github.com/user-attachments/assets/02d7bc6f-2a1d-4b7e-818d-9b4f30631caa)

![image](https://github.com/user-attachments/assets/5e4cd91d-0f0f-46e0-a265-ba2e90acf0bc)

"Title" is an extra field, you can leave it empty, if you want.

I don't really like this note type. It's like Cloze, but with multiple answers. I believe this isn't beneficial since it makes recall much easier than cloze, which isn't good for strengthening memories, and the only "advantage" is that it looks fancy. Just use Cloze, or even better - Randomized Cloze.


## Alternatives

There is another note type that's conceptually similar to Match Pairs: https://github.com/cjdduarte/anki-template-interactive-drag-drop/blob/main/Example.apkg. Feel free to use whichever you like more.

___
### [←Return to homepage](https://expertium.github.io/)
