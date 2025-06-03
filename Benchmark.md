# Benchmark of Spaced Repetition Algorithms

# Table of contents
- [Intro](#intro)
- [Metrics](#metrics)
- [Algorithms](#algorithms)
  - [DSR memory model](#dsr-memory-model)
  - [Other memory models](#other-memory-models)
  - [Neural networks](#neural-networks)
  - [Miscellaneous algorithms](#miscellaneous-algorithms)
- [Dataset](#dataset)
  - [Time Series Split](#time-series-split) 
- [Results](#results)
  - [Log loss, RMSE and AUC](#log-loss-rmse-and-auc)
  - [Superiority](#superiority)
- [Discussion](#discussion)
- [References](#references)


## Intro
This is an extended version of my Reddit post. This article should take approximately 28–38 minutes to read, the Reddit post should take around 9–13 minutes.
Side note: when I say "we", I'm referring to myself and [Jarrett Ye](https://github.com/L-M-Sherlock), the creator of [FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS).


## Metrics
First of all, every spaced repetition algorithm must be able to predict the **probability of recalling** a card at a given point in time, given the card's review history. Let's call that **R**. All three metrics that we use are related to the probability of recall.

If an algorithm doesn't calculate probabilities and just outputs an interval, it's still possible to convert that interval into a probability under certain assumptions. It's better than nothing, since it allows us to perform at least some sort of comparison. That's what we did for SM-2, the only algorithm in the entire benchmark that wasn't originally designed to predict probabilities. We decided not to include [Memrise](https://memrise.zendesk.com/hc/en-us/articles/360015889057-How-does-the-spaced-repetition-system-work) because we are unsure if the assumptions required to convert its intervals to probabilities hold. Well, it wouldn't perform great anyway, it's about as inflexible as possible.

Once we have an algorithm that predicts R, we can run it on some users' review histories to see how much predicted R deviates from measured R. If we do that using hundreds of millions of reviews, we will get a very good idea of which algorithm performs better on average. **RMSE** (bins), or root mean square error, can be interpreted as "the average difference between predicted and measured probability of recall (R)". RMSE (bins) is a measure of **calibration**. I will keep adding (bins) just to make it clear that it's not the same as the standard RMSE.

Loosely speaking, if an algorithm predicts an X% chance of something happening, it should happen X% of the time. For example, if a weatherman says, "There is a 90% chance of rain today,"  it should rain on 90% of days when he says that. That's good calibration. If it rained only on 40% of those days, it means that the weatherman (or, well, his forecasting system) is poorly calibrated​  -  ​his probabilities don't match observed frequencies. RMSE (bins) measures how well-calibrated an algorithm is.

The calculation of RMSE (bins) has been reworked in the past to reduce cheating, aka algorithms achieving good numbers on paper without getting better at predicting R in reality. If you want to know the nitty-gritty mathematical details, you can read [this article by Jarrett and me](https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric). The new method is our own invention, and you won't find it in any academic paper. Anki >=24.04 uses the new method when "Evaluate" is used.

Here's what you need to know about RMSE (bins):
1. RMSE (bins) measures how close the predicted R is to reality. It puts reviews into bins and measures the difference between *average* actual retention and *average* predicted R within each bin.
2. RMSE (bins) ranges from 0 to 1, lower is better.
3. We calculate it by binning reviews in a way that is specific to spaced repetition, you won't find this in the literature. This isn't very nice for making the benchmark results accepted by other researchers, but we have other metrics for that.
4. It's not what the optimizer in Anki is minimizing. Log loss is what the FSRS optimizer is internally using for optimization, not RMSE (bins). We can't use RMSE (bins) for that.

Next is log loss. It is calculated using the following formula:

![CodeCogsEqn (10)](https://github.com/user-attachments/assets/c38fecc4-0f35-42e4-a02b-86756410e641)

where *y* is a binary label (either 0 or 1; 0 = Again and 1=Hard/Good/Easy in the context of Anki), and *R* is the probability of recall (a real number between 0 and 1) predicted by some algorithm.

Here's what you need to know about log loss:
1. Log loss measures how close the predicted probability of recall (R) is to reality, just like RMSE (bins). However, unlike RMSE (bins), it doesn't rely on binning reviews. RMSE (bins) is based on the difference between averages, whereas log loss is based on the difference between individual predictions and individual review outcomes.
2. Log loss ranges from 0 to infinity, lower is better.
3. Unlike RMSE (bins), log loss never reaches 0, unless the algorithm only outputs ones and zeros. If an algorithm outputs numbers between 0 and 1, the minimum possible value of log loss that it can achieve is >0. This makes log loss less intuitive than RMSE (bins). You might intuitively expect that if the distribution of predicted probabilities exactly matches the distribution of true probabilities, the loss would be zero, but no.

Next is AUC (Area Under the Curve). Unlike the previous two metrics, AUC is not a measure of **calibration** but of **discrimination**. Here's what you need to know about AUC:
1. AUC measures how well an algorithm can tell classes apart; in our case, classes are "recalled" and "forgotten." You can think of AUC as a measure of how well the algorithm can draw a boundary between two classes, such that all members of class 1 are on one side of the boundary and all members of class 2 are on the other side.
2. AUC score ranges from 0 to 1, but in practice it's almost always greater than 0.5. An AUC score less than 0.5 indicates that the algorithm is performing worse than random. Higher is better.

AUC can be rather unintuitive in some cases. Exaggerated example: suppose you have an algorithm that always outputs a 99% probability of having cancer for people who do have cancer and a 98% probability of having cancer for people who do not have cancer. It never outputs 98% for those who do have cancer, and it never outputs 99% for those who don't. What do you think is the AUC score of this algorithm? Answer: 1.0, because it can perfectly distinguish between these two classes, even if the calibration is absolutely terrible. AUC doesn't tell us anything about calibration, only about discrimination.

Below is a diagram that explains AUC.

![AUC 2 1](https://github.com/user-attachments/assets/28c55552-0613-4643-ad69-bfecf04dfce8)

For a more in-depth explanation of AUC, you can read [this article](https://www.geeksforgeeks.org/auc-roc-curve/). The important part is how to interpret it: in the context of spaced repetition, the AUC score can be interpreted as "a probability that the algorithm will assign a higher probability of recall to a recalled card than to a forgotten card". For example, AUC=0.7 means that there is a 70% chance that a recalled card has a higher probability of recall predicted by the algorithm than a forgotten card.


Here's a table comparing different metrics.

![Comparison table (metrics)](https://github.com/user-attachments/assets/42aeaf91-0a5b-43bf-92e6-23f1c88c42c7)


## Algorithms

### DSR memory model

Most of the algorithms are based on the Stability, Retrievability (alternatively Half-Life, Probability) model of memory, or it's extension, Difficulty, Stability, Retrievability (alternatively Difficulty, Half-Life, Probability). I will refer to the former as the SR model and to the latter as the DSR model.

All FSRS algorithms use the DSR model of memory.

1.​ ​FSRS v3. It was the first version of FSRS that people actually used, it was released in October 2022. It wasn't terrible, but it had issues. Jarrett, I, and several other users have proposed and tested several dozens of ideas (only a handful of them proved to be effective) to improve the algorithm. ​FSRS v1 and v2 were initial experimental versions of FSRS that were only used by Jarrett. I did not include them here.

2.​ FSRS v4. It came out in July 2023, and at the beginning of November 2023, it was integrated into Anki. It's a significant improvement over v3.

3.​ FSRS-4.5. It's a slightly improved version of FSRS v4, the shape of the forgetting curve has been changed.

4.​ FSRS-5. It has 2 more parameters than FSRS-4.5 and it takes into account same-day reviews, unlike all previous versions; though the improvement in accuracy is small.

5.​ FSRS-6. The newest version. It has 2 more parameters than FSRS-5. One for same-day reviews, and oen for controlling the shape of the forgetting curve. Before FSRS-6 the shape was the same for all users. In this article I show results for FSRS-6 with recency weighting. Recency weighting makes more recent reviews have greater importance during optimization, meaning that FSRS adapts more to newer reviews at the cost of adapting les to older reviews.

6.​ FSRS-6 (default parameters). This is just to see how well FSRS-6 performs without optimization. To see how FSRS has evolved over time, check this out: [https://imgur.com/a/calibration-of-different-fsrs-versions-KfJ32EV](https://imgur.com/a/calibration-of-different-fsrs-versions-KfJ32EV). Althought I recommend looking at those graphs *after* reading the rest of the article.

Below is a diagram that should give you a better understanding of FSRS. If you want to know the details, please read [this article](/Algorithm.md).

![FSRS (proper)](https://github.com/user-attachments/assets/44f568d8-afce-4a49-a782-99531fcc352c)

"Grade" refers to Again/Hard/Good/Easy. To calculate the loss, the grade is converted into a binary value: 0 if it's Again, 1 otherwise. This doesn't mean that FSRS *itself* treats grades as binary, of course it can tell the difference between Hard, Good and Easy. Loss is calculated only during optimization (this is true for all diagrams you will see), so if you want to imagine how FSRS works when it's deployed and is not being optimized, just mentally remove the part about loss computation. Note that p(recall) and retrievability are the same thing.

In order to calculate the probability of recall, FSRS requires the length of the previous interval and its own previous state, which is represented using three numbers: Difficulty, memory Stability, and Retrievability (DSR). Notice that horizontal arrows always point to the right, showing that past states can affect future states, but future states cannot affect past states.

7.​ [HLR](https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf), Half-Life Regression. It's an algorithm developed by Duolingo for Duolingo. The memory half-life in HLR is conceptually very similar to the memory stability in FSRS, but it's calculated using an overly simplistic formula. It uses the SR model.

![HLR (proper)](https://github.com/user-attachments/assets/1532b41c-25bf-4c74-a6bf-5ff1f6b7abf9)

For HLR, the order of reviews doesn't matter because it only requires summary statistics about the whole review history. Regardless of how you rearrange reviews, the total number of reviews, passed reviews, and failed reviews (lapses) will remain the same.

In Duolingo, HLR also incorporates linguistic data into the model, which we don't have in Anki, so in Duolingo it likely performs a little better. A little.

8.​ Ebisu v2. [It's an algorithm that uses Bayesian statistics](https://fasiha.github.io/ebisu/) to update its estimate of memory half-life after each review. While it has 3 parameters, Ebisu was not designed to optimize them automatically, they have to be configured manually. It uses the SR model.

### Other memory models

These algorithms are based on a different model, not SR or DSR.

9.​ [DASH](https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m), Difficulty, Ability and Study History. This is an actual *bona fide* model of human memory based on neuroscience. Well, kind of. The issue with it is that the forgetting curve looks like a step function. There are also DASH[MCM] and DASH[ACT-R], but I won't inlcude them in this article. You can read more [here](https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf).

![DASH (proper)](https://github.com/user-attachments/assets/e678dd4a-536b-4631-a26b-a0bce04ffa67)

(ok, I admit, this diagram is a mess, but I don't know how to make it clearer)

DASH, DASH[MCM] and DASH[ACT-R] don't have state variables that are carried on between reviews, and they don't process reviews sequentially, like SM-17/18 or FSRS. Instead, all past reviews must be processed in order to calculate the length of the next interval.

10.​ [ACT-R](http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf), Adaptive Control of Thought​  -  ​Rational (I've also seen "Character" instead of "Control" in some papers). It's based on a model of human memory that makes one very strange assumption: whether you have successfully recalled your material or not doesn't affect the magnitude of the spacing effect, only the interval length matters. Simply put, this algorithm doesn't differentiate between Again/Hard/Good/Easy. Notice that in the diagram below, grades are only used for calculating the loss function during optimization, but not used by the algorithm itself - no arrows come from "Grade". 

![ACT-R (proper)](https://github.com/user-attachments/assets/0ed88ec8-b5bd-41da-b3ee-8c7f1bd5bcd8)

Below are some example forgetting curves.

![Forgetting curves](https://github.com/user-attachments/assets/5f791191-6721-4406-a854-a0c53da36a0b)

The Y axis doesn't start at 0.

These curves were plotted using default parameters, which have been obtained by running each algorithm on ~~20~~ 10 thousand collections of Anki users. So what you're seeing are "average" or "typical" curves.
DASH's curve looks like a step function, which goes against our human intuition and common sense. DASH[MCM] attempts to smooth it, but you can see that it's not perfect. DASH[ACT-R] achieves a smooth curve. <br />
Also, the probability of recall doesn't start at 100% for DASH algorithms and ACT-R. <br />

### Neural networks

11.​ GRU-P (GRU stands for Gated Recurrent Unit). This neural network architecture is commonly used for time series analysis, such as predicting stock market trends or recognizing human speech. This implementation uses the SR model - it calculates memory stability as an intermediate variable.

![GRU (proper)](https://github.com/user-attachments/assets/aed193fe-0b48-49a7-93df-9bd447da490f)

GRU is also a recurrent algorithm, just like FSRS, even if the mathematical formulas are completely different. Its state is represented by an array with n numbers, where n is called the dimension of the hidden state. In this implementation n=2.

Unlike GRU, which predicts memory stability before converting it into R via a power forgetting curve formula, GRU-P predicts R directly. This implementation does not rely on SR or DSR models of memory. In this implementation n, the dimension of the hidden state, is 8. GRU-P (short-term) also uses same-day reviews, so it's trained on more data. I only included GRU-P (short-term) for the sake of not making this article any longer and not making the graphs more cluttered.

12.​ LSTM. This is also a recurrent neural network, but a more complex one than GRU. I won't make a diagram for it. This implementation calculates three different values of memory stability for three different forgetting curve and then combines them into one via weighted averaging (with learnable weights). It uses same-day reviews and, unlike most algorithms here, it uses fractional interval lengths. It also uses the answer time - how long the user spent on a card - as an input feature. It was pretrained using the [Reptile algorithm](https://openai.com/index/reptile/).

13.​ [RWKV](https://github.com/BlinkDL/RWKV-LM) (pronounced "rʌkuv" in IPA, like "tho<ins>**rou**</ins>gh <ins>**k**</ins>ettle m<ins>**ove**</ins>"), a novel architecture that aims to combine the best of Transformers and recurrent neural nets. It has too many input features to list, so here is a *short* version: fractional interval lengths, grades, duration of the review, note ID, deck ID, preset ID, sibling card information, hour of the day, day of the week, and the number of reviews done today.

Unlike other algorithms in this benchmark, RWKV is not optimized on each user individually. Instead, it is optimized on 5 thousand users and evaluated on another 5 thousand; this process is repeated twice to get full coverage of the dataset. All other algorithms are optimized on a per-user basis, meaning that the parameters are personalized for each user, and the algorithm is evaluated on the review history of *the same user* that it's trained on, just on a different, later part of that history. This is explained in more detail in the [Time Series Split](#time-series-split) section. Huge thanks to [1DWalker](https://github.com/1DWalker) on Github for implementing RWKV and LSTM!

14.​ RWKV-P. Same idea as with GRU-P: no forgetting curve in the traditional sense, it predicts the probability of recall directly, which can lead to unintuitive results. It's technically the same neural net as RWKV without P, it can work in two different "regimes". I decided to include both RWKV and RWKV-P to show the difference in performance, but keep in mind that RWKV and RWKV-P are actually *exactly* the same neural net, just used in different ways.

### Miscellaneous algorithms

15.​ [Anki SM-2](https://faqs.ankiweb.net/what-spaced-repetition-algorithm#sm-2). A variant of SM-2 used in Anki. ISM-2 is a 35+ year-old algorithm that is (with some changes) still used by Anki, Mnemosyne, and possibly other apps as well. It's main advantage is simplicity. We put a not-so-rigorous interval-to-probability converter on top of it because **it was not originally designed to predict probabilities**.

16.​ AVG. It's an "algorithm" that outputs a constant equal to the user's average retention. For example, if the user presses Hard/Good/Easy 85% of the time, the "algorithm" will always output an 85% probability of recall for any given review. You can think of it as a weatherman who says, "The temperature today will be average, the wind speed will be average, and the humidity will be average as well" every single day. This "algorithm" is intended only to serve as a baseline for comparison and has no practical applications. It does not use any model of memory. If an algorithm does not outperform AVG, it cannot be considered good.

Some variations of these algorithms are not included because this list is already a bit too big and the article is already a bit too long.

I did my best to create a "taxonomy" of spaced repetition algorithms.

![All algorithms (new) 3 1](https://github.com/user-attachments/assets/4d9d8b19-bcf5-440c-bd1c-56166122a9ae)

SM-2 is not included in this diagram because it wasn't designed to predict the probability of recall, unlike the other algorithms.
SM-17/18 algorithms also use the three-component (Difficulty, Stability, Retrievability) model of memory.
HLR lacks a difficulty variable, but it does have memory stability (half-life) and retrievability, so one could say that it employs a two-component model of memory rather than a three-component model.


## Dataset

The dataset used in the benchmark is [Anki revlogs 10k](https://huggingface.co/datasets/open-spaced-repetition/anki-revlogs-10k), the largest in the world. It contains data about ~727 million flashcard reviews from 10 thousand users, which is approximately 3 times more reviews than in the [Maimemo dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0) and approximately 56 times more reviews than in [the Duolingo dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME).

The data itself is also different. In Anki, each user makes their own flashcards, while Maimemo and Duolingo offer pre-made courses. Simply put, Anki has "same learner - different material" kind of data, and Maimemo/Duolingo has "same material - different learner" kind of data.

This benchmark is based on 9,999 collections and 349,923,850 reviews. Same-day reviews are excluded except when optimizing FSRS-5 and algorithms that have "-short" at the end of their names, which use same-day reviews for training but not for evaluation. Additionally, some reviews are filtered out, such as when the user manually changed the due date (which would count as a review) or when the user used what's called a "filtered deck" if "Reschedule cards based on my answers in this deck" was disabled. Finally, an outlier filter is applied. Because of all of that, the real number of reviews used for evaluation is around 350 million, much smaller than the 727 million figure mentioned above. Data is split into training sets and test sets. Algorithm are trained on training sets and evaluated on test sets.

### Time Series Split

If you want a more technical explanation of how the data is split, here:

1​.​ Data is split into 5 parts, let's call them A-B-C-D-E. A contains the oldest reviews, E contains the most recent reviews.

2​.​ An algorithm is trained on A and evaluated on B. Let's call the error that is obtained at this step error 1.

3​.​ An algorithm is trained on A and B and evaluated on C to obtain error 2.

4​.​ An algorithm is trained on A, B and C and evaluated on D to obtain error 3.

5​.​ An algorithm is trained on A, B, C and D and evaluated on E to obtain error 4.

6​.​ The final error is a simple average of four errors, and the final parameters are from step 5.

This procedure is used for all algorithms except for RWKV. Thanks to this clever way of splitting data, no data is thrown away while at the same time the algorithm is trained on different data than it is evaluated on, so we get a realistic estimate of how it would perform on new, previously unseen data.

By the way, this is not how "Evaluate" works in Anki. "Evaluate" uses a simplified procedure to avoid optimizing parameters every time the user wants to evaluate them - it just evaluates the specified parameters on the entire history, without optimizing them and without any splitting; training set=test set. "Evaluate" can only tell the user how well the parameters fit the *current* review history aka the training set. But the benchmark should evaluate the algorithm's ability to generalize beyond the training set data. In Anki 25.06 "Evaluate" has been removed.

I hope this diagram helps:

![How benchmarking is done 3 1](https://github.com/user-attachments/assets/a4d0084b-12d4-4591-86e5-56d76d4d8565)

Split A is not used for evaluation, split E is not used for training.

Some algorithms, like LSTM and RWKV, use more information than just interval lengths and grades, for example answer time. Using other *input features* is fine as long as evaluation is done on the *same reviews of the same cards*. Of course, the evaluation of all algorithms is based not just on exactly the same total number of reviews (349,923,850), but on the exactly the *same* reviews.


## Results

### Log loss, RMSE and AUC

In the tables and charts below, the averages are weighted by the number of reviews in each user's collection, meaning that users with more reviews have a greater impact on the value of the average. If someone has 100 thousand reviews, they will affect the average 100 times more than someone who only has 1 thousand reviews.
The tables also show the number of optimizable parameters of each algorithm. The benchmark repo also has [unweighted averages](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#result).

![RMSE table](https://github.com/user-attachments/assets/eef66b64-9678-49b9-9ceb-eb0bdfe2d2ba)

![RMSE](https://github.com/user-attachments/assets/1b4417a7-3d38-4e31-ad91-13adb95993c1)

Lower is better. Black caps are 99% confidence intervals.
Don't focus too much on absolute values, they depend on a lot of things: how we calculate RMSE (which involves somewhat arbitrary binning), whether the averages are weighted by the number of reviews or not, and how the outlier filter works. Instead, focus on the ranking​  -  ​which algorithm is the best, which one is the second best, which one is the third best, etc.

Now let's look at log loss. Reminder: both log loss and RMSE (bins) measure how close predicted probability of recall is to reality.

![Log loss table](https://github.com/user-attachments/assets/6bba1278-ff93-4772-b69c-f49620834750)

![Log loss](https://github.com/user-attachments/assets/d91b7ff9-e2e1-48a4-9088-258f15c38725)

Lower is better. Black caps are 99% confidence intervals.
As you can see, the ranking is a little different.

Finally, let's look at AUC scores. Reminder - higher scores indicate better ability to differentiate between recalled and forgotten cards. Scores close to 0.5 indicate that the algorithm is no better than chance at telling apart recalled and forgotten cards.

![AUC table](https://github.com/user-attachments/assets/17de696c-711d-4c62-bd33-802ef28d51a6)

![AUC](https://github.com/user-attachments/assets/a0f961f4-6e91-48a1-8f97-ac702934e0d8)

Higher is better. Black caps are 99% confidence intervals.  <br />
Now ranking is very different.
It's interesting that the AUC score of HLR is 0.6333, much higher than 0.54, which is what Duolingo reported in their paper. Granted, 0.6333 is not that impressive either. In fact, all spaced repetition algorithms have rather unimpressive AUC scores. <br />
Surprisingly, AVG has an AUC score slightly above 0.5. Since it always outputs a constant, it cannot differentiate between forgotten and recalled cards, so in theory, it should have an AUC score of *exactly* 0.5. <br />

On all three graphs if an algorithm is to the right of AVG, it is not good. AVG always outputs a constant equal to the user's average retention. If a spaced repetition algorithm cannot outperform a constant prediction, it cannot be considered good.

As you can see, RWKV outperforms FSRS according to all 3 metrics, and by a very big margin. So a more general algorithm that can effectively leverage large amounts of compute has outperformed an algorithm that was hand-crafted by domain experts. [Hmmm, sounds oddly familiar, where have I heard that before?](https://www.cs.utexas.edu/~eunsol/courses/data/bitter_lesson.pdf)


### Superiority

The metrics presented above can be difficult to interpret. In order to make it easier to understand how algorithms perform relative to each other, the image below shows the percentage of users for whom algorithm A (row) has a lower RMSE (bins) than algorithm B (column). For example, GRU-P-short has a 96.1% superiority over Ebisu v2, meaning that for 96.1% of all collections in this benchmark, GRU-P-short can estimate the probability of recall more accurately than Ebisu.

![Superiority-small-9999-collections](https://github.com/user-attachments/assets/069776be-001a-46a3-b92b-9c0fecbe7ff6)

You may have noticed that FSRS-6 (with recency weighting) has a 99.6% superiority over Anki SM-2, meaning that for 99.6% of users, log loss will be lower with FSRS-6 than with SM-2. But please remember that SM-2 wasn’t designed to predict probabilities, and the only reason it does that in this benchmark is because extra formulas for converting intervals given by SM-2 into probabilities were added on top of it. **There is no way to have a truly fair, no caveats, comparison between FSRS and SM-2.**

Here are a few fun numbers from this diagram:

FSRS-6 (recency) vs FSRS-5: 88.2% superiority.

FSRS-6 (recency) optimized vs FSRS-6 with default parameters: 84.3% superiority. <br />
You may be thinking, "Wait, so in ~16% of cases, default parameters are better? That seems too high." The reason is that optimization and evaluation are performed on different data. This is a common practice in machine learning. Evaluating the performance of an algorithm on the same data that it was trained on usually leads to an overly optimistic estimate of performance, and in reality the algorithm performs worse. To get a more realistic estimate, the algorithm is trained on one subset of data (training set) and evaluated on another one (test set). Informally, you can think of it like giving a student practice problems as homework but evaluating him based on his answers during the exam rather than based on his answers to homework problems. <br />
So what this really means is not "in 16% of cases, default parameters fit the data better", but "in 16% of cases, FSRS fails to generalize beyond the training data sufficiently well and doesn't perform well on new, unseen data". This is more likely to happen if the amount of data (reviews) is low, and less likely to happen for old, large collections.

You may also wonder why RWKV-P is so much better than RWKV. Is it because the idea of a forgetting curve is bogus? No. If you want a smooth and monotonic forgetting curve, you have to give up the ability to change predictions after they have been made. Suppose that today FSRS predicted that the stability of this card is 365 days, and at 90% desired retention, it will show it to you in a year. A year has passed. In the absence of optimization, the card still the same S=365 days, which will only be updated once a card is reviewed. FSRS cannot say "You know, I think memory stability of this card is more like 330 days actually, I was off, can I change my prediction?". RWKV can't either. But RWKV-P is not constrained by that. In other words, RWKV-P sacrifices smoothness and monotonicity for the ability to adjust it's predictions as new information comes in even if the parameters are frozen aka no optimization.

And just out of curiosity, here is a comparison of forgetting curves of FSRS-6 with recency weighting and RWKV.

![image](https://github.com/user-attachments/assets/287da038-c4ba-499e-b18e-7f2a8bae7979)

![image](https://github.com/user-attachments/assets/24617225-0fe1-4a2b-8ac3-cf4ed16c7b72)

Green vertical line = a passing review (Hard/Good/Easy).

Notice that RWKV predicts that probability of recall falls off very sharply after the first review, within hours or even minutes, but then declines much more slowly. FSRS doesn't have a short-term memory model, so it cannot predict something like that.


## Discussion

Caveats:

1. We cannot benchmark proprietary algorithms, such as the latest SuperMemo algorithms.

2. There are algorithms that require extra features, such as HLR with Duolingo's lexeme tags or [KAR3L](https://arxiv.org/pdf/2402.12291.pdf), which uses not only interval lengths and grades but also the text of the card and mildly outperforms FSRS v4 (though it's unknown whether it outperforms FSRS-4.5, FSRS-5 and FSRS-6), according to the paper. Such algorithms can be more accurate than FSRS when given the necessary information, but they cannot be benchmarked on our dataset. Only algorithms that use interval lengths and grades can be benchmarked since no other features are available.

We would love to benchmark [THLR](https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling), but the researchers didn't release their code publicly.

Regarding the future of FSRS, we have been racking our brains, trying to come up with some way to improve it. **FSRS-6 is the final version, there will be no major releases in the foreseeable future.**

Broadly speaking, machine learning algorithms are bound by the amount of computational power available, by the amount of data, and by the software. FSRS is not bound by computational power at all, its parameters can be optimized on an average home PC in a matter of seconds; training FSRS for 10x as long would only improve the metrics by 1-2%. FSRS is somewhat bound by data since most users don't have hundreds of thousands of reviews. And it's almost entirely bound by software, aka the theory of memory and forgetting.

There are several ways to make FSRS more accurate, none of which are currently feasible:

1. Consider the number of reviews done before a particular review and the time of day to estimate how fatigued the user was (perhaps some other factors could be taken into account when estimating fatigue as well). If someone is doing their first review at 4 PM, they are probably less fatigued than someone who is doing their 500th review at 4 AM, which affects retention. This is not possible with the way Anki currently works​  -  ​FSRS cannot access datetime information​  -  ​and would require major changes.

2. Consider the content of the cards: text, sound, and images. It would require adding another machine learning algorithm (or even several algorithms) just for text/audio/image recognition, and we wouldn't be able to train it since Dae (the main Anki dev) can't give us a dataset that has all of the content of cards. That is against Anki's privacy policy, only scheduling data is available publicly.

3. Consider the interference from sibling cards. This could also be extended to "conceptual siblings"​  -  ​cards that don't come from the same note, but test you on similar material. Again, not possible due to how Anki works: when the user is reviewing a particular card, FSRS (or any other algorithm) cannot access the review history of any other card. So it cannot combine data from multiple cards. Even if it could, optimization would probably become a nightmare.

All three of these combined could greatly increase the efficiency of spaced repetition. 2 and 3 combined could be particularly effective if each pair of cards is assigned a "similarity score" (using some machine learning algorithm) based on their content, though doing that naively would be computationally intractable​  -  ​the number of pairs is proportional to the number of cards squared; for example, 10,000 cards have 49,995,000 pairs. Still, I would expect great improvements from an algorithm that doesn't treat cards as independent, and a review of card A affects not only the memory stability of card A but also the memory stability of card B, albeit to a lesser extent.

**Anki is not designed for advanced spaced repetition algorithms.** <br />
There are about 20 different ways to get learning steps wrong, and having two arbitrary stages ("learning" and "review") isn't necessary to begin with. <br />
Any algorithm, FSRS or otherwise, can only access interval lengths and grades, nothing else. <br />
Datetime information is inaccessible when scheduling the next review. <br />
Information from other cards (other than the card that is being reviewed right now) is inaccessible when scheduling the next review. <br />
[There is no way to manually create connections between cards](https://faqs.ankiweb.net/linking-cards-together.html). <br />

That being said, let's imagine a future where RWKV-P was successfully integrated into Anki. Here's what it would look like:
1. No "Optimize" button, RWKV would be pretrained on 10k users and then the same parameters would be used for everyone.
2. No parameters window.
3. Accurate probability of recall for any interval, even on the scale of minutes and seconds, unlike FSRS.
4. No user-defined learning steps. Instead, there would probably just be a "Enable same-day reviews" toggle.
5. RWKV can accurately predict R for cards for which it is **impossible** for FSRS to perform well. Consider the following simplified example: the user was in a good mood and was spamming Good at first, but then his mood got worse, and now he is spamming Again. The sequence looks like this: [Good, Good, Good, Good, Good, Again, Again, Again, Again, Again]. FSRS cannot take advantage of this pattern, RWKV-P can.
6. No memory stability and difficulty values, and also no forgetting curve graphs.
7. No intervals above answer buttons. Instead, scheduling would be completely different: every hour/minute/second RWKV-P would calculate R for all cards, then it would show you cards for which R is below the threshold that is desired retention. You can’t really calculate intervals in a meaningful way using RWKV-P. So instead it would just recalculate R once per hour/minute/second and show you what needs to be reviewed. It would be extremely computationally expensive to do this every minute (let alone every second), so for the foreseeable future this is not viable.

## References

References to academic papers:

1. [https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m](https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m) (DASH is first mentioned on page 68)
2. [https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf](https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf)
3. [http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf](http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf)
4. [https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf](https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf)
5. [https://arxiv.org/pdf/2402.12291.pdf](https://arxiv.org/pdf/2402.12291.pdf)
6. [https://www.cs.utexas.edu/~eunsol/courses/data/bitter_lesson.pdf](https://www.cs.utexas.edu/%7Eeunsol/courses/data/bitter_lesson.pdf)
7. [https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling](https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling)

References to things that aren't academic papers:

1. [https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#srs-benchmark)
2. [https://www.geeksforgeeks.org/auc-roc-curve/](https://www.geeksforgeeks.org/auc-roc-curve/)
3. [https://huggingface.co/datasets/open-spaced-repetition/anki-revlogs-10k](https://huggingface.co/datasets/open-spaced-repetition/anki-revlogs-10k)
4. [https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric](https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric)
5. [https://supermemo.guru/wiki/Algorithm_SM-17](https://supermemo.guru/wiki/Algorithm_SM-17)
6. [https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0)
7. [https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME)
8. [https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/](https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/)
9. [https://github.com/BlinkDL/RWKV-LM](https://github.com/BlinkDL/RWKV-LM)


___
### [←Return to homepage](https://expertium.github.io/)
