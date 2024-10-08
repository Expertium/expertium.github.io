# JReadability

# Intro

The authors of [this paper](https://researchmap.jp/jhlee/published_papers/21426109) present a simple formula for calculating "readability" of Japanese texts. Higher scores indicate that this is a text for beginners, lower scores indicate that this is an advanced text.
This is intended to help Japanese learners select appropriate material for their level. This is not intended for native Japanese speakers.

The Python implementation of their formula can be found here: https://github.com/joshdavham/jreadability. However, I was left somewhat unsatisfied and felt that a better model can be made rather easily. Thanks to Josh, I was able to get my hands on the data from https://cijapanese.com/, specifically,
transcripts of videos classified as Complete Beginner, Beginner, Intermediate, and Advanced. Then me and Josh wrote down every feature that could be relevant to estimating readability.

## Text Features

1) Average sentence length. The longer the sentence, the less likely a beginner is to fully grasp its meaning.
2) Average frequency rank of kanji used in the text. In case you don't know what "frequency rank" means, here's an example: according to my own custom frequency list, 日 is the most commonly used kanji in the Japanese language, which means its frequency rank is 1. 年 is the second most commonly used kanji, which means its rank is 2, etc.
3) Mean number of commas in a sentence. Simple sentences don't have commas.
4) Proportion of wago (漢語): words of Japanese origin.
5) Proportion of kango (漢語): words of Chinese origin.
6) Proportion of verbs (動詞).
7) Proportion of adverbs (副詞).
8) Proportion of nouns (名詞).
9) Proportion of katakana words aka loanwords.
10) Proportion of particles (助詞).
11) Proportion of determinants.
12) Proportion of subordinating conjuctions.

## Model

Then I made a simple linear model where the number of parameters is equal to the number of features plus one (because of the constant). And in order to see how much benefit there is in adding more features, I tested 7 versions of the model, including the original version proposed in "Readability measurement of Japanese texts based on levelled corpora."

Below is a table comparing all of the different versions:

Parameters were optimized by minimizing the Root Mean Square Error (RMSE). I converted levels to numbers in the following way: Complete Beginner = 4, Beginner = 3, Intermediate = 2, Advanced = 1. Error = (label-prediction)^2. RMSE is calculated as the square root of the average of squared errors.
I also calculated the Spearman rank correlation between predictions and labels. Spearman's correlation coefficient is more appropriate here than Pearson's because Spearman's works better when one of the variables is ordinal (as opposed to continuous).

Below are graphs illustrating Spearman's correlation coefficients and average RMSE of each version:

And here is the formula used in JReadability-12, the best version that uses all 12 aforementioned features:

## Examples

Japanese text: おはようございます！今日は天気がいいですね。This means "Good morning! The weather is nice today". Readability score = .

Japanese text: 船員は自衛隊員が務め、観測隊員は厳しい審査と訓練に合格した人間だけ。This means "The boat is manned by the military, and the expedition members have to pass strict screenings and training". Readability score = .

## Implementation

From now on, https://github.com/joshdavham/jreadability is using my JReadability-12 to output a readability score between 1 and 4. It has been implemented in the JReadability add-on for Anki. Huge thanks to [Josh](https://github.com/joshdavham) for helping me obtain the dataset and to ___ for making the add-on!


___
### [←Return to homepage](https://expertium.github.io/)
