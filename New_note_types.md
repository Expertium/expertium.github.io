# New note types

[https://ankiweb.net/shared/info/171015247](https://ankiweb.net/shared/info/171015247). The deck has examples of 4 new note types: Match Pairs, Randomized Cloze, Randomized Basic and Click Words.

- [Match Pairs](#match-pairs)
- [Randomized Cloze](#randomized-cloze)
- [Randomized Basic](#randomized-basic)
- [Click Words](#click-words)

## Match Pairs

Have you ever had cards like this? There are 2 pieces of knowledge and you can't remember which is which, so you make a Cloze.

![1](https://github.com/user-attachments/assets/def904c9-b78b-437f-ac66-d8f0807f155f)

But there is a problem - you may end up just memorizing "thingy 1 is the top one, thingy 2 is the bottom one". In order to avoid that, you could make two notes with the order switched.

![2](https://github.com/user-attachments/assets/8ef6dd0a-6417-4106-a422-10048bd67af6)

However, this is inefficient - now you have 2 notes for the same thing. If only there was a way to put them into the same note and randomize order...

Well, with Match Pairs there is!


![ezgif-7-6213ee4eb8](https://github.com/user-attachments/assets/8b2ef2dd-1211-4ddd-9e36-d50dad5cd14c)

And it's not the only advantage of this note type. It also supports images.

![Images](https://github.com/user-attachments/assets/5edef924-016f-46c1-aa11-a53f4e708ed2)

And audio (if you can't hear it in the example below, make sure to click the speaker symbol ![image](https://github.com/user-attachments/assets/7c5e85ac-1357-4ac3-a68b-c2a65e0d2877)).

https://github.com/user-attachments/assets/6f65fb06-322b-4745-8ee9-101b126e2df5

And if you think that this is too easy, and therefore would make active recall ineffective, you can make your life harder by adding 1 or 2 fake answers.

![3 capitals](https://github.com/user-attachments/assets/4133a134-4501-4a7b-a17b-a562d0ec3228)

Here you have 2 countries but 3 capitals, so you can't answer correctly if you only know the capital of one country but not the capital of the other one. <br />
Make sure that the extra answer is wrong, but not *obviously* wrong. In this example, I won't benefit from adding Jakarta to the second list, since it's obviously wrong. Which is why I added Amsterdam. I don't have to think very hard to remember that Jakarta is neither the capital of Sweden nor Switzerland, whereas Amsterdam makes me pause and think.

Btw, you don't necessarily have to drag answers - you can click on them. When you click on an answer, it is put in the *top* box.

![Answer is put at the top](https://github.com/user-attachments/assets/4c1fbea7-7f69-4a34-81c1-d0e86637d2dc)

Here's what the fields look like:

![image](https://github.com/user-attachments/assets/87a2b1b6-231b-40c2-8934-b0f9977b1cd8)

`|` is the separator that you should put between items. Don't worry about leading/trailing spaces, they are stripped away automatically: `Answer1 | Answer2` will produce the same result as `Answer1|Answer2`.


When you download the deck, there will be this card:

![image](https://github.com/user-attachments/assets/ff46142b-776b-479a-bf9a-884e76761ef3)

As it says, don't delete it. The thing about audio is that it's impossible to play multiple audio files in whatever order you want. This is a problem for Match Pairs, where the order in which udio plays should, ideally, be randomized. Solution: just don't let Anki play the audio automatically, so that the user has to play it manually. You can do that by using this setting:

![image](https://github.com/user-attachments/assets/1708ee2d-c553-4233-acb4-472be5d0cb0d)

But that will affect other note types as well. So some crude hacks are needed to prevent only Match Pairs from playing audio automatically.


Of course, how useful this note type is for you depends on how often you encounter what I call "negative interference", where card A makes it harder to remember card B, and card B makes it harder to remember card A. Personally, I really wish that this note type would become a built-in type. I think it has enough advantages to justify being a part of Anki.

## Randomized Cloze

This is another note type that aims to solve the "memorizing the shape of the question rather than the content" problem.

![Randomized Cloze](https://github.com/user-attachments/assets/75c665cc-470a-4930-b527-ef1e586ab04b)

To save some time and effort, you can ask ChatGPT, Claude or Gemini to rephrase the sentence and generate 2-3 sentences with the same meaning, although I recommend taking the time to write sentences yourself.

One thing that you should keep in mind: the numbers in curly brackets have to be the same, otherwise you'll end up making multiple cards instead of one card. The separator is the same as for Match Pairs.

![Cloze edit](https://github.com/user-attachments/assets/5881adc5-7150-4f3a-8efa-90517425eb7a)

## Randomized Basic

It's exactly what it sounds like. And the separator is the same.



## Click Words

[The deck with these new note types](https://ankiweb.net/shared/info/171015247) also has a fourth note type, but I don't really like it. Feel free to try it out on your own.


## Alternatives

There is another note type that's conceptually similar to Match Pairs: https://github.com/cjdduarte/anki-template-interactive-drag-drop/blob/main/Example.apkg. Feel free to use whichever you like more.

___
### [←Return to homepage](https://expertium.github.io/)
