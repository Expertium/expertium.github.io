# JReadability

## Intro

The authors of ["Introducing a readability evaluation system for Japanese language education"](https://jreadability.net/file/hasebe-lee-2015-castelj.pdf) and ["Readability measurement of Japanese texts based on levelled corpora"](https://researchmap.jp/jhlee/published_papers/21426109) present a simple formula for calculating "readability" of Japanese texts. Higher scores indicate that this is a text for beginners, lower scores indicate that this is an advanced text.
This is intended to help Japanese learners select appropriate material for their level. This is not intended for native Japanese speakers.

The Python implementation of their formula can be found here: https://github.com/joshdavham/jreadability. However, I was left somewhat unsatisfied and felt that a better model can be made rather easily. Thanks to Josh, I was able to get my hands on the data from https://cijapanese.com, specifically,
transcripts of videos classified as Complete Beginner, Beginner, Intermediate, and Advanced. The dataset consists of 231 transcripts of videos labeled as Complete Beginner, 315 Beginner video transcripts, 287 Intermediate video transcripts and 77 Advanced video transcripts. In total, there are 910 texts and 2,022,948 characters after removing spaces. I thought the dataset is somewhat imbalanced, so I added 150 more Advanced texts: 66 from random Wikipedia articles, 30 from scientific papers and 54 from books from the Aozora Bunko digital library. The final enhanced dataset contains 1,060 texts and 2,175,630 characters.

I am not allowed to share the dataset, apologies to the few data scientists who would like to try their own models. But you can register at https://cijapanese.com and the email them, maybe they'll share the dataset with you, too.

Then me and Josh wrote down every feature that could be relevant to estimating readability.

## Text features

1) Mean sentence length. The longer the sentence, the less likely a beginner is to fully grasp its meaning.
2) Mean frequency rank of kanji used in the text. In case you don't know what "frequency rank" means, here's an example: according to my own custom frequency list, 日 is the most commonly used kanji in the Japanese language, which means its frequency rank is 1. 年 is the second most commonly used kanji, which means its rank is 2, etc. Simple texts have common kanji, sophisticated texts have more obscure kanji.
3) Mean frequency rank of words used in the text. This is similar to the previous one, but also different, since a word can be made of several kanji or no kanji at all. The kanji frequency dictionary and the word frequency dictionary are based on different data and made by different people who probably used different approaches, which isn't ideal, but it's the best we have.
4) Mean number of relative repetitions. Ok, I don't fully understand this one, this was Josh's idea. Basically, it's a measure of how many words are repeated several times.
5) Percentage of wago (和語): words of Japanese origin.
6) Percentage of kango (漢語): words of Chinese origin.
7) Percentage of verbs (動詞).
8) Percentage of adverbs (副詞).
9) Percentage of nouns (名詞), excluding proper nouns and numerals, those are counted separately.
10) Percentage of katakana words aka loanwords.
11) Percentage of particles (助詞).
12) Percentage of aixiliary verbs (助動詞).
13) Percentage of interjections (感動詞).
14) Percentage of proper nouns (固有名詞).
15) Percentage of personal pronouns.
16) Percentage of adjectives (形容詞).
17) Percentage of numerals (数詞).
18) Percentage of coordinating conjunctions (接続詞).
19) Percentage of symbols, such as $, %, §, ©, +, −, ×, ÷, =, <, >, etc.

Thanks to [fugashi](https://pypi.org/project/fugashi/), most of it can be done very simply. I only had to make functions for 1, 2, 8 and 13 on my own.

## The model

I made a simple linear model where the number of parameters is equal to the number of features plus one (because of the constant). And in order to see how much benefit there is in adding more features, I tested 8 versions of the model, including the original version proposed in "Readability measurement of Japanese texts based on levelled corpora." Each version is called JReadability-X, where X is the number of features used in the formula. The number of parameters is X+1.

Parameters were optimized using [sklearn.linear_model.LinearRegression()](https://scikit-learn.org/1.5/modules/generated/sklearn.linear_model.LinearRegression.html). I converted levels to numbers in the following way: Complete Beginner = 4, Beginner = 3, Intermediate = 2, Advanced = 1. 

For evaluation of the goodness-of-fit, I used two metrics: Spearman's rank correlation coefficient, and RMSE is calculated as the square root of the average of squared errors. Spearman's correlation coefficient is more appropriate here than Pearson's because Spearman's works better when one of the variables is ordinal (as opposed to continuous).

![RMSE](https://github.com/user-attachments/assets/3428265b-46f7-4491-b858-13f23340a159)


Below is a table comparing all of the different versions:

![image](https://github.com/user-attachments/assets/d60d0651-ca1d-4f98-8e31-50eb8a1761ff)

And here are graphs illustrating Spearman's correlation coefficients and average RMSE of each version:

![JReadability RMSE](https://github.com/user-attachments/assets/70ef6eba-4bb0-4b39-985a-60272e0acc43)

![JReadability Spearman](https://github.com/user-attachments/assets/b3e7ac10-bfaf-4848-a96e-4c26db65eca7)


## Examples

Japanese text: "おはようございます". This means "Good morning!". Readability score = 4.00.

Japanese text: "船員は自衛隊員が務め、観測隊員は厳しい審査と訓練に合格した人間だけ". This means "The ship is manned by the military, and the expedition members have to pass strict screenings and training". Readability score = 1.91.

## Implementation

https://github.com/___ is using my JReadability-12 to output a readability score between 1 and 4. It has been implemented in the [JReadability add-on]() for Anki. Huge thanks to [Josh](https://github.com/joshdavham) for helping me obtain the dataset and to ___ for making the add-on! And make sure to read Josh's article [on readability](https://cij-analysis.streamlit.app/).


___
### [←Return to homepage](https://expertium.github.io/)
