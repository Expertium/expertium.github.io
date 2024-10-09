# JReadability

## Intro

The authors of ["Introducing a readability evaluation system for Japanese language education"](https://jreadability.net/file/hasebe-lee-2015-castelj.pdf) and ["Readability measurement of Japanese texts based on levelled corpora"](https://researchmap.jp/jhlee/published_papers/21426109) present a simple formula for calculating "readability" of Japanese texts. Higher scores indicate that this is a text for beginners, lower scores indicate that this is an advanced text.
This is intended to help Japanese learners select appropriate material for their level. This is not intended for native Japanese speakers.

The Python implementation of their formula can be found here: https://github.com/joshdavham/jreadability. However, I was left somewhat unsatisfied and felt that a better model can be made rather easily. Thanks to Josh, I was able to get my hands on the data from https://cijapanese.com/, specifically,
transcripts of videos classified as Complete Beginner, Beginner, Intermediate, and Advanced. The dataset consists of 231 transcripts of videos labeled as Complete Beginner, 315 Beginner video transcripts, 287 Intermediate video transcripts and 77 Advanced video transcripts. In total, there are 910 texts and 2,071,441 characters.

Then me and Josh wrote down every feature that could be relevant to estimating readability.

## Text features

1) Mean sentence length. The longer the sentence, the less likely a beginner is to fully grasp its meaning.
2) Mean frequency rank of kanji used in the text. In case you don't know what "frequency rank" means, here's an example: according to my own custom frequency list, 日 is the most commonly used kanji in the Japanese language, which means its frequency rank is 1. 年 is the second most commonly used kanji, which means its rank is 2, etc. Simple texts have common kanji, sophisticated texts have more obscure kanji.
3) Percentage of wago (和語): words of Japanese origin.
4) Percentage of kango (漢語): words of Chinese origin.
5) Percentage of verbs (動詞).
6) Percentage of adverbs (副詞).
7) Percentage of nouns (名詞), excluding proper nouns and numerals, those are counted separately.
8) Percentage of katakana words aka loanwords.
9) Percentage of particles (助詞).
10) Percentage of aixiliary verbs (助動詞).
11) Percentage of interjections (感動詞).
12) Percentage of proper nouns (固有名詞).
13) Percentage of personal pronouns.
14) Percentage of adjectives (形容詞).
15) Percentage of numerals (数詞).
16) Percentage of coordinating conjunctions (接続詞).
17) Percentage of symbols, such as $, %, §, ©, +, −, ×, ÷, =, <, >, etc.

Thanks to [fugashi](https://pypi.org/project/fugashi/), most of it can be done very simply. I only had to make functions for 1, 2, 8 and 13 on my own.

## The model

I made a simple linear model where the number of parameters is equal to the number of features plus one (because of the constant). And in order to see how much benefit there is in adding more features, I tested 8 versions of the model, including the original version proposed in "Readability measurement of Japanese texts based on levelled corpora." Each version is called JReadability-X, where X is the number of features used in the formula. The number of parameters is X+1.

Below is a table comparing all of the different versions:

Parameters were optimized by minimizing the Root Mean Square Error (RMSE), I used `scipy.optimize.minimize`. I converted levels to numbers in the following way: Complete Beginner = 4, Beginner = 3, Intermediate = 2, Advanced = 1. RMSE is calculated as the square root of the average of squared errors.

![RMSE](https://github.com/user-attachments/assets/ecc3e586-e8f5-40fb-b847-02a8abdf7069)

I also calculated the Spearman rank correlation between predictions and levels. Spearman's correlation coefficient is more appropriate here than Pearson's because Spearman's works better when one of the variables is ordinal (as opposed to continuous).

Below are graphs illustrating Spearman's correlation coefficients and average RMSE of each version:

And here is the formula used in JReadability-16, the best version that uses all 16 aforementioned features:

## Examples

Japanese text: おはようございます！今日は天気がいいですね。This means "Good morning! The weather is nice today". Readability score = .

Japanese text: 船員は自衛隊員が務め、観測隊員は厳しい審査と訓練に合格した人間だけ。This means "The ship is manned by the military, and the expedition members have to pass strict screenings and training". Readability score = .

## Implementation

https://github.com/___ is using my JReadability-12 to output a readability score between 1 and 4. It has been implemented in the [JReadability add-on]() for Anki. Huge thanks to [Josh](https://github.com/joshdavham) for helping me obtain the dataset and to ___ for making the add-on!


___
### [←Return to homepage](https://expertium.github.io/)
