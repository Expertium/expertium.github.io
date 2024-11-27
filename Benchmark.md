# Benchmark of Spaced Repetition Algorithms

# Table of contents
- [Intro](#intro)
- [Metrics](#metrics)
- [Algorithms](#algorithms)
- [Dataset](#dataset)
- [Results](#results)
- [Discussion](#discussion)
- [References](#references)

## Intro
This is an extended version of my Reddit post. This article should take approximately 27–32 minutes to read, the Reddit post should take around 11–13 minutes. I tried to make this article suitable for both tech-savvy and casual readers, so I may have over-explained some things.
Side note: when I say "we", I'm referring to myself and [Jarrett Ye](https://github.com/L-M-Sherlock), the creator of [FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS).


## Metrics
First of all, every spaced repetition algorithm must be able to predict the **probability of recalling** a card at a given point in time, given the card's review history. Let's call that **R**. All three metrics that we use are related to the probability of recall.

If an algorithm doesn't calculate probabilities and just outputs an interval, it's still possible to convert that interval into a probability under certain assumptions. It's better than nothing, since it allows us to perform at least some sort of comparison. That's what we did for SM-2, the only algorithm in the entire benchmark that wasn't originally designed to predict probabilities. We decided not to include [Memrise](https://memrise.zendesk.com/hc/en-us/articles/360015889057-How-does-the-spaced-repetition-system-work) because we are unsure if the assumptions required to convert its intervals to probabilities hold. Well, it wouldn't perform great anyway, it's about as inflexible as possible.

Once we have an algorithm that predicts R, we can run it on some users' review histories to see how much predicted R deviates from measured R. If we do that using hundreds of millions of reviews, we will get a very good idea of which algorithm performs better on average. **RMSE**, or root mean square error, can be interpreted as "the average difference between predicted and measured probability of recall (R)". RMSE is a measure of **calibration**.

Loosely speaking, if an algorithm predicts an X% chance of something happening, it should happen X% of the time. For example, if a weatherman says, "There is a 90% chance of rain today,"  it should rain on 90% of days when he says that. That's good calibration. If it rained only on 40% of those days, it means that the weatherman (or, well, his forecasting system) is poorly calibrated​  -  ​his probabilities don't match observed frequencies. RMSE measures how well-calibrated an algorithm is.

The calculation of RMSE has been reworked in the past to prevent cheating, aka algorithms achieving good numbers on paper without getting better at predicting R in reality. If you want to know the nitty-gritty mathematical details, you can read [this article by Jarrett and me](https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric). The new method is our own invention, and you won't find it in any academic paper. Anki >=24.04 uses the new method when "Evaluate" is used.

Here's what you need to know about RMSE:
1. RMSE measures how close the predicted R is to reality. It puts reviews into bins and measures the difference between *average* actual retention and *average* predicted R within each bin.
2. RMSE ranges from 0 to 1, lower is better.
3. We calculate it by binning reviews in a way that is specific to spaced repetition, you won't find this in the literature. This isn't very nice for making the benchmark results accepted by other researchers, but we have other metrics for that.
4. It's not what the optimizer in Anki is minimizing. Log loss is what the FSRS optimizer is internally using for optimization, not RMSE. We can't use RMSE for that.

Next is log loss. It is calculated using the following formula:

![CodeCogsEqn (10)](https://github.com/user-attachments/assets/c38fecc4-0f35-42e4-a02b-86756410e641)

where *y* is a binary label (either 0 or 1), and *R* is the probability of recall (a real number between 0 and 1) predicted by some algorithm.

Here's what you need to know about log loss:
1. Log loss measures how close the predicted probability of recall (R) is to reality, just like RMSE. However, unlike RMSE, it doesn't rely on binning reviews. RMSE is based on the difference between averages, whereas log loss is based on the difference between individual predictions and individual review outcomes.
2. Log loss ranges from 0 to infinity, lower is better.
3. Unlike RMSE, log loss never reaches 0, unless the algorithm only outputs ones and zeros. If an algorithm outputs numbers between 0 and 1, the minimum possible value of log loss that it can achieve is >0. This makes log loss less intuitive than RMSE. You might intuitively expect that if the distribution of predicted probabilities exactly matches the distribution of true probabilities, the loss would be zero, but no.

Next is AUC (Area Under the Curve). Unlike the previous two metrics, AUC is not a measure of **calibration** but of **discrimination**. Here's what you need to know about AUC:
1. AUC measures how well an algorithm can tell classes apart; in our case, classes are "recalled" and "forgotten." You can think of AUC as a measure of how well the algorithm can draw a boundary between two classes, such that all members of class 1 are on one side of the boundary and all members of class 2 are on the other side.
2. AUC scores range from 0 to 1, but in practice they're almost always greater than 0.5. AUC scores less than 0.5 indicate that the algorithm is performing worse than random. Higher is better.

AUC can be rather unintuitive in some cases. Exaggerated example: suppose you have an algorithm that always outputs a 99% probability of having cancer for people who do have cancer and a 98% probability of having cancer for people who do not have cancer. It never outputs 98% for those who do have cancer, and it never outputs 99% for those who do. What do you think is the AUC score of this algorithm? Answer: 1.0, because it can perfectly distinguish between these two classes, even if the calibration is absolutely terrible. AUC doesn't tell us anything about calibration, only about discrimination.

Below is a diagram that explains AUC.

![AUC 2 1](https://github.com/user-attachments/assets/28c55552-0613-4643-ad69-bfecf04dfce8)

For a more in-depth explanation of AUC, you can read [this article](https://www.geeksforgeeks.org/auc-roc-curve/).


Here's a table comparing different metrics.

![Comparison table (metrics)](https://github.com/user-attachments/assets/42aeaf91-0a5b-43bf-92e6-23f1c88c42c7)


## Algorithms

Most of the algorithms are based on the Stability, Retrievability (alternatively Half-Life, Probability) model of memory, or it's extension, Difficulty, Stability, Retrievability (alternatively Difficulty, Half-Life, Probability). I will refer to the former as the SR model and to the latter as the DSR model.

### FSRS family

All FSRS algorithms use the DSR model of memory.

1.​ ​FSRS v3. It was the first version of FSRS that people actually used, it was released in October 2022. It wasn't terrible, but it had issues. Jarrett, I, and several other users have proposed and tested several dozens of ideas (only a handful of them proved to be effective) to improve the algorithm.

2.​ FSRS v4. It came out in July 2023, and at the beginning of November 2023, it was integrated into Anki. It's a significant improvement over v3.

3.​ FSRS-4.5. It's a slightly improved version of FSRS v4, the shape of the forgetting curve has been changed.

4.​ FSRS-5. The newest version. The main difference is that it takes into account same-day reviews, unlike all previous versions, though the improvement in performance is small. Anki 24.XX uses it, AnkiMobile and AnkiDroid haven't been updated yet, so they use FSRS-4.5. If you want to read about the differences between FSRS-4.5 and FSRS-5, I have an article with an in-depth explanation of FSRS formulas.

5.​ FSRS-5 (default parameters). This is just to see how well FSRS-5 performs without optimization.

6.​ FSRS-5 (pretrain). In FSRS, the first 4 parameters (values of initial stability) are optimized in a completely different way compared to the rest. "Pretrain" is when the first 4 parameters are optimized, while the rest of parameters are set to default. In Anki >=24.06, when parameters are optimized, the optimizer determines whether to keep the default parameters, perform pretrain, or perform a full optimization; which one is used depends on the user's review history. The more reviews a user has, the more likely it is that full optimization will be performed.

Below is a diagram that should give you a better understanding of FSRS. If you want to know the details, please read [this article](/Algorithm.md).

![FSRS](https://github.com/user-attachments/assets/89dd0ca5-6579-4471-bf95-2b1c3535f881)

In order to calculate the length of the next interval, FSRS requires the length of the previous interval, grade (Again/Hard/Good/Easy) and its own previous state, which is represented using three numbers: Difficulty, memory Stability, and Retrievability (DSR). Notice that horizontal arrows always point to the right, showing that past states can affect future states, but future states cannot affect past states.

### General-purpose machine learning algorithms family

7.​ ormer. This neural network architecture has become popular in recent years because of its superior performance in natural language processing. ChatGPT uses this architecture. This implementation uses the SR model.

8.​ GRU, Gated Recurrent Unit. This neural network architecture is commonly used for time series analysis, such as predicting stock market trends or recognizing human speech. Originally, we used a more complex architecture called LSTM, but GRU performed better with fewer parameters. Both GRU and ormer use the same power forgetting curve as FSRS-4.5 and FSRS-5 to make the comparison more fair. This implementation uses the SR model.

![GRU](https://github.com/user-attachments/assets/49f3152b-524f-46d3-b202-4b0090f921d0)

GRU is also a recurrent algorithm, just like FSRS, even if the mathematical formulas are completely different. Its state is represented by one number.

9.​ GRU-P. Unlike GRU, which predicts memory stability before converting it into R via a power forgetting curve formula, GRU-P predicts R directly. More about GRU-P later. This implementation does not rely on SR or DSR models of memory.

10.​ GRU-P (short-term). Same as above, but it also uses same-day reviews, so it's trained on more data. This implementation does not rely on SR or DSR models of memory.

### DASH family

These algorithms are based on a different model, not SR or DSR.

11.​ [DASH](https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m), Difficulty, Ability and Study History. This is an actual *bona fide* model of human memory based on neuroscience. Well, kind of. The issue with it is that the forgetting curve looks like a step function.

12.​ DASH[MCM]. A hybrid model, it addresses some of the issues with DASH's forgetting curve.

13.​ DASH[ACT-R]. Another hybrid model, it finally achieves a smooth forgetting curve.

[Here](https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf) is another relevant paper.

![DASH](https://github.com/user-attachments/assets/b0b0b3c7-998e-4c77-8955-f9cb650bc180)

DASH, DASH[MCM] and DASH[ACT-R] don't have state variables that are carried on between reviews, and they don't process reviews sequentially, like SM-17/18 or FSRS. They are more like ormers: all past reviews must be processed in order to calculate the length of the next interval. This makes them much slower than FSRS when the number of reviews is large. In FSRS, each review takes a constant amount of time to process. If a card has 100 reviews, processing the first review will take the same amount of time as processing the 100th review. In DASH, the processing time of a single review depends on the number of past reviews. Therefore, processing the 100th review takes much longer than processing the first one.

Also, even though the diagram shows "Next interval length" as output, in reality, calculating the next interval using DASH algorithms would be very difficult due to the quirks of their forgetting curves. With FSRS and HLR, calculating the probability of recall that corresponds to a specific interval length and calculating the interval length that corresponds to a specific probability of recall is equally easy, but not with DASH algorithms. Still, I drew the diagram this way because a diagram that shows how algorithms predict probabilities would be much harder to read.

### Other algorithms

14.​ [ACT-R](http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf), Adaptive Control of Thought​  -  ​Rational (I've also seen "Character" instead of "Control" in some papers). It's a model of human memory that makes one very strange assumption: whether you have successfully recalled your material or not doesn't affect the magnitude of the spacing effect, only the interval length matters. Simply put, this algorithm doesn't differentiate between Again/Hard/Good/Easy.

![ACT-R](https://github.com/user-attachments/assets/f566d8d2-9d33-4f19-868f-313b9dfd2866)

Below are some example forgetting curves.

![Forgetting curves 2 1](https://github.com/user-attachments/assets/08694d4b-fdfa-49cb-93d0-7712ffe2e71a)

The Y axis doesn't start at 0.

These curves were plotted using default parameters, which have been obtained by running each algorithm on 20 thousand collections of Anki users. So what you're seeing are "average" or "typical" curves.
DASH's curve looks like a step function, which goes against our human intuition and common sense. DASH[MCM] attempts to smooth it, but you can see that it's not perfect. DASH[ACT-R] achieves a smooth curve. <br />
Also, the probability of recall doesn't start at 100% for DASH models and ACT-R. <br />
It's interesting that the forgetting curve of FSRS-4.5 (and FSRS-5, they use the same formula) is so steep compared to other models. FSRS v3 used a much steeper exponential formula, which was replaced with a less steep power formula in FSRS v4, and with an even less steep power formula in FSRS-4.5. And yet, even that still predicts much faster forgetting than other models. While we could make the forgetting curve of FSRS-5 even less steep, it would practically prevent the probability of recall from ever reaching values less than 10%, since even for small values of memory stability, it would take more than a human life to reach 10% with such a curve.

15.​ [HLR](https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf), Half-Life Regression. It's an algorithm developed by Duolingo for Duolingo. The memory half-life in HLR is conceptually very similar to the memory stability in FSRS, but it's calculated using an overly simplistic formula. It uses the SR model.

![HLR](https://github.com/user-attachments/assets/5bfd0b0a-d030-4889-a478-27a8f520cbd9)

For HLR, the order of reviews doesn't matter because it only requires summary statistics about the whole review history. Regardless of how you rearrange reviews, the total number of reviews, passed reviews, and failed reviews (lapses) will remain the same.

16.​ SM-2. It's a 35+ year-old algorithm that is still used by Anki, Mnemosyne, and possibly other apps as well. It's main advantage is simplicity. Note that in our benchmark, it is implemented the way it was originally designed. It's not the Anki version of SM-2, it's the original SM-2. We put a not-so-rigorous interval-to-probability converter on top of it.

17.​ NN-17. It's a neural network approximation of [SM-17](https://supermemo.guru/wiki/Algorithm_SM-17). The SuperMemo wiki page about SM-17 may appear very detailed at first, but it actually obfuscates all of the important details that are necessary to implement SM-17. It tells you what the algorithm is doing, but not how. Our approximation relies on the limited information available on the formulas of SM-17 while utilizing neural networks to fill in any gaps. It uses teh DSR model.

![NN-17](https://github.com/user-attachments/assets/f877e8f9-f06c-46c7-9573-335bfccb196b)

In order to calculate the length of the next interval, NN-17 requires the length of the previous interval, grade (Again/Hard/Good/Easy) and its own previous state, which is represented using four numbers: Difficulty, memory Stability, Retrievability, and the number of lapses.

18.​ Ebisu v2. [It's an algorithm that uses Bayesian statistics](https://fasiha.github.io/ebisu/) to update its estimate of memory half-life after each review. While it has 3 parameters, Ebisu was not designed to optimize them automatically, they have to be configured manually. It uses the SR model.

19.​ AVG. It's an "algorithm" that outputs a constant equal to the user's average retention. For example, if the user presses Hard/Good/Easy 85% of the time, the "algorithm" will always output an 85% probability of recall for any given review. You can think of it as a weatherman who says, "The temperature today will be average, the wind speed will be average, and the humidity will be average as well" every single day. This "algorithm" is intended only to serve as a baseline for comparison and has no practical applications. It does not use any model of memory.

I did my best to create a "taxonomy" of spaced repetition algorithms.

![All algorithms (new) 1 1](https://github.com/user-attachments/assets/f629c0f6-d324-489c-a5b1-f9afeeac74df)

SM-2 is not included in this diagram because it wasn't designed to predict the probability of recall, unlike the other algorithms.
SM-17/18 algorithms also use the three-component (Difficulty, Stability, Retrievability) model of memory.
HLR lacks a difficulty variable, but it does have memory stability (half-life) and retrievability, so one could say that it employs a two-component model of memory rather than a three-component model.

Additionally, below is a table comparing different algorithms.

![Comparison table 1 2](https://github.com/user-attachments/assets/ec4377aa-76b4-4eb5-9750-b0a695024e92)

It should be noted that some algorithms, such as SM-2, which were not designed to be adaptive, could still be made adaptive and benefit from parameter optimization. In the table, boxes in the Adaptive column are checked based on the original design of the algorithm.

Regarding invertibility: for some algorithms, such as DASH and GRU-P, probabilities can be calculated easily, but due to the quirks of their forgetting curves, it is not possible to determine interval lengths that correspond to specific probabilities via an exact formula. Though it can still be done approximately. Nonetheless, invertibility is desirable for practical purposes of scheduling.


## Dataset

The dataset used in the benchmark is [FSRS Anki 20k](https://huggingface.co/datasets/open-spaced-repetition/FSRS-Anki-20k), the largest in the world. It contains data about ~1.7 billion flashcard reviews from 20 thousand users, which is approximately 8 times more reviews than in the [Maimemo dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0) and approximately 131 times more reviews than in [the Duolingo dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME).

The data itself is also different. In Anki, each user makes their own flashcards, while Maimemo and Duolingo offer pre-made courses. Anki has data about certain users reviewing different material, while Maimemo and Duolingo have data about certain material being reviewed by different users.

This benchmark is based on 19,990 collections and 702,721,850 reviews, excluding same-day reviews (which constitute approximately 24.6% of all reviews); only 1 review of a card per day is used by each algorithm other than FSRS-5 and GRU-P (short-term), which use same-day reviews for training but not for evaluation. Additionally, some reviews are filtered out, such as when the user manually changed the due date (which would count as a review) or when the user used what's called a "filtered deck" if "Reschedule cards based on my answers in this deck" was disabled. Finally, an outlier filter is applied. Because of all of that, the real number of reviews used for evaluation is around 700 million, much smaller than the 1.7 billion figure mentioned above.


## Results

In the tables and charts below, the averages are weighted by the number of reviews in each user's collection, meaning that users with more reviews have a greater impact on the value of the average. If someone has 100 thousand reviews, they will affect the average 100 times more than someone who only has 1 thousand reviews.
The tables also show the number of optimizable parameters of each algorithm. The benchmark repo also has [unweighted averages](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#result).

![image](https://github.com/user-attachments/assets/939b8604-1287-4752-a3ee-8ef97367ed8f)

![RMSE](https://github.com/user-attachments/assets/7774c392-322c-4304-be18-46b0b1f72573)

Lower is better. Black caps are 99% confidence intervals.
Don't focus too much on absolute values, they depend on a lot of things: how we calculate RMSE (which involves somewhat arbitrary binning), whether the averages are weighted by the number of reviews or not, and how the outlier filter works. Instead, focus on the ranking​  -  ​which algorithm is the best, which one is the second best, which one is the third best, etc.

The bars corresponding to FSRS-5 and GRU-P (short-term) are colored differently to indicate that these algorithms have been trained on more reviews; all other algorithms were trained without using same-day reviews. However, the comparison is still fair because all algorithms are evaluated on the same data, even if the training data is different. Here's an analogy: one student did a lot of homework before the test, the other did less homework. After that, both students took the same test. Even though they did a different amount of homework, since the test is the same, it's still valid to compare the test scores of these two students.

Now let's look at log loss.

![image](https://github.com/user-attachments/assets/acae828d-6a4a-48d8-a218-2bb2fd9e0bf0)

![Log loss](https://github.com/user-attachments/assets/5fcc05a2-db57-4e28-bfa8-ec8d7e1af452)

Lower is better. Black caps are 99% confidence intervals.
As you can see, the ranking is a little different. For example, based on RMSE, the ranks of NN-17 and GRU are very close (10th and 11th best, respectively), but based on log loss, NN-17 is ranked much higher (9th best, GRU is 14th). SM-2 and Transformer have switched places.

Finally, let's look at AUC.

![image](https://github.com/user-attachments/assets/b079375e-250c-4499-9f7f-07b779bd7bf3)

![AUC](https://github.com/user-attachments/assets/44772a1c-c0f4-4601-87dd-7742b04d2951)

Higher is better. Black caps are 99% confidence intervals.  <br />
Now ranking is very different. This isn't too surprising, considering that AUC is completely uncorrelated with both RMSE and log loss. <br />
It's interesting that the AUC score of HLR is 0.631, much higher than 0.54, which is what Duolingo reported in their paper. Granted, 0.631 is not that impressive either. In fact, all spaced repetition algorithms have rather unimpressive AUC scores. <br />
Unsurprisingly, AVG has an AUC score close to 0.5. Since it always outputs a constant, it cannot differentiate between forgotten and recalled cards. <br />
It is somewhat surprising that NN-17 has a relatively low AUC score, given that it combines the best of both worlds​  -  ​a model of human memory supplemented with a neural network. Granted, the goal  was not to create the perfect algorithm; rather, the goal was to emulate SM-17. <br />
Jarrett's implementation of Transformer doesn't perform well according to all 3 metrics, so if any neural network experts think, "I bet I can do better!" they are welcome. I think it's probably because each algorithm is only trained for 5 epochs, which is more than enough for FSRS and other simple models, but not enough for complex models.

Let's address GRU-P. As you can see, it outperforms all other algorithms by all three metrics. So you're probably wondering "If predicting R directly is better than predicting an intermediate value first, why not do that?". Here's what happens when you let an algorithm predict R directly.

![GRU-P curves](https://github.com/user-attachments/assets/3420e64b-8bfb-4533-89f4-77908877af66)

These are forgetting curves that GRU-P generated for different users. Only one of them makes sense. <br />
A curve that becomes flat (top left) not only makes no sense but is also unusable in practice, it could result in *infinitely* long intervals when used for scheduling. <br />
A curve with a maximum that is not at time=0 (bottom left) makes no sense either. A curve with a minimum (top right) implies that after some point in time, forgetting ends and some sort of anti-forgetting starts, which also makes no sense. <br />
Only the bottom right is a proper forgetting curve. Well, minus the fact that it's wiggly. And minus the fact that it doesn't start at 100%. <br />
So while GRU-P outperforms all other algorithms, it's not usable in practice as it could result in all kinds of strange behavior.

Notice that while GRU-P (short-term) outperforms GRU-P and while FSRS-5 outperforms FSRS-4.5, the difference in all 3 metrics is very small. This suggests that **same-day reviews have a very small impact on long-term memory**. Since the architecture of FSRS and GRU-P is very different, the fact that the improvement is small for both of them suggests that architecture is not to blame here.

You might be thinking, "But what if the dataset just has very few same-day reviews? Then it would appear that, on average, their impact is small." That's a valid concern, but in the Anki 20k dataset, 24.6% of reviews are same-day reviews (this is *after* excluding manual due date changes and other special cases). So clearly, lack of data isn't an issue.

One more thing. The metrics presented above can be difficult to interpret. In order to make it easier to understand how algorithms perform relative to each other, the image below shows the percentage of users for whom algorithm A (row) has a lower RMSE than algorithm B (column). For example, GRU-P-short has a 94.5% superiority over the Transformer, meaning that for 94.5% of all collections in this benchmark, GRU-P-short can estimate the probability of recall more accurately than the Transformer.

![Superiority, 19990](https://github.com/user-attachments/assets/eee84fd8-0820-40ab-a2da-4e309cb8a6c9)

You may have noticed that FSRS-5 has a 99.0% superiority over SM-2, meaning that for 99.0% of users, RMSE will be lower with FSRS-5 than with SM-2. But please remember that SM-2 wasn’t designed to predict probabilities, and the only reason it does that in this benchmark is because extra formulas for converting intervals given by SM-2 into probabilities were added on top of it. **There is no way to have a truly fair, no caveats, comparison between FSRS and SM-2.**

Here are a few fun numbers from this diagram:

FSRS-5 vs FSRS-4.5: 67.9% superiority.

FSRS-5 vs FSRS v4: 82.5% superiority.

FSRS-5 vs FSRS v3: 90.4% superiority.

FSRS-5 optimized vs FSRS-5 with default parameters: 81.8% superiority. <br />
You may be thinking, "Wait, so in 18% of cases, default parameters are better? That seems too high." The reason is that optimization and evaluation are performed on different data. This is a common practice in machine learning. Evaluating the performance of an algorithm on the same data that it was trained on usually leads to an overly optimistic estimate of performance, and in reality the algorithm performs worse. To get a more realistic estimate, the algorithm is trained on one subset of data (training set) and evaluated on another one (test set). Informally, you can think of it like giving a student practice problems as homework but evaluating him based on his answers during the exam rather than based on his answers to homework problems. <br />
So what this really means is not "in 18% of cases, default parameters fit the data better", but "in 18% of cases, FSRS fails to generalize beyond the training data sufficiently well and doesn't perform well on new, unseen data". This is more likely to happen if the amount of data (reviews) is low, and less likely to happen for old, large collections.

If you want a more technical explanation, here:

1​.​ Data is split into 5 parts, let's call them A-B-C-D-E. A contains the oldest reviews, E contains the most recent reviews.

2​.​ An algorithm is trained on A and evaluated on B. Let's call the error that is obtained at this step error 1.

3​.​ An algorithm is trained on A and B and evaluated on C to obtain error 2.

4​.​ An algorithm is trained on A, B and C and evaluated on D to obtain error 3.

5​.​ An algorithm is trained on A, B, C and D and evaluated on E to obtain error 4.

6​.​ The final error is a simple average of four errors, and the final parameters are from step 5.

This procedure is used for all algorithms. Thanks to this clever way of splitting data, no data is thrown away while at the same time the algorithm is trained on different data than it is evaluated on, so we get a realistic estimate of how it would perform on new, previously unseen data.

By the way, this is not how "Evaluate" works in Anki. "Evaluate" uses a simplified procedure to avoid optimizing parameters every time the user wants to evaluate them - it just evaluates the specified parameters on the entire history, without optimizing them and without any splitting; training set = test set. "Evaluate" can only tell the user how well the parameters fit the *current* review history aka the training set. But the benchmark should evaluate the model's ability to generalize beyond the training set data. Jarrett believes that it's fine that the benchmark and Anki don't use the same evaluation procedure.



## Discussion

Caveats:

1. We cannot benchmark proprietary algorithms, such as the latest SuperMemo algorithms.

2. There are algorithms that require extra features, such as HLR with Duolingo's lexeme tags or [KAR3L](https://arxiv.org/pdf/2402.12291.pdf), which uses not only interval lengths and grades but also the text of the card and mildly outperforms FSRS v4 (though it's unknown whether it outperforms FSRS-4.5 and FSRS-5), according to the paper. Such algorithms can be more accurate than FSRS when given the necessary information, but they cannot be benchmarked on our dataset. Only algorithms that use interval lengths and grades can be benchmarked since no other features are available.

We would love to benchmark [THLR](https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling), but the researchers didn't release their code publicly.

Regarding the future of FSRS, we have been racking our brains, trying to come up with some way to improve it, and this mild improvement in FSRS-5 was the best we could do. **FSRS-5 is the final version, there will be no major releases in the foreseeable future.**

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

With all that in mind, I want to make several predictions:

1​.​ No further version of FSRS beyond FSRS-5 will be used in Anki by 2027. No FSRS-5.5, FSRS-6, or any other version that supersedes FSRS-5.
Clarification: I made this prediciton a few days before Jarrett made [this tweet](https://x.com/JarrettYe/status/1817570865699299818). After seeing his tweet, I'm even more confident in this prediction. I have an idea for FSRS-6, but it requires getting a new dataset and using more input features than just interval lengths and grades. Also, Jarrett said that unless some other famous app decides to implement FSRS, he won't work on FSRS-6 just for the sake of Anki. Overall, I find it unlikely that FSRS-6 will be released before 2027.

~~2​. By 2029, no algorithm in our benchmark will have achieved a (weighted by the number of reviews) log loss lower than 0.27, unless the dataset used in the benchmark changes, in which case this prediction is rendered void.~~

~~3​. By 2029, no algorithm in our benchmark will have achieved an (weighted by the number of reviews) AUC score higher than 0.83, unless the dataset used in the benchmark changes.~~

4​.​ By 2031, there will be an app with an algorithm that employs at least one out of the three ideas proposed above (which are not specific to FSRS), and that app will not be Anki. For example, an app using KAR3L.
The app must be publicly available in AppStore, Google Play Store, or elsewhere, and it must not be in the beta testing stage. I'm adding these extra conditions because, without them, [mathacademy.com](https://www.mathacademy.com/) has already [met the main condition](https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/). Even with the extra conditions, this prediction can easily come true way sooner than 2031.

Predictions were made at the end of July, 2024.

The dataset has changed, the new dataset includes deck IDs and preset IDs, so predictions 2 and 3 will have to be revised.


## References

References to academic papers:

1. [https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m](https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m) (DASH is first mentioned on page 68)
2. [https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf](https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf)
3. [http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf](http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf)
4. [https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf](https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf)
5. [https://arxiv.org/pdf/2402.12291.pdf](https://arxiv.org/pdf/2402.12291.pdf)
6. [https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling](https://www.researchgate.net/publication/381792698_DRL-SRS_A_Deep_Reinforcement_Learning_Approach_for_Optimizing_Spaced_Repetition_Scheduling)

References to things that aren't academic papers:

1. [https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#srs-benchmark)
2. [https://www.geeksforgeeks.org/auc-roc-curve/](https://www.geeksforgeeks.org/auc-roc-curve/)
3. [https://huggingface.co/datasets/open-spaced-repetition/FSRS-Anki-20k](https://huggingface.co/datasets/open-spaced-repetition/FSRS-Anki-20k)
4. [https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric](https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-Metric)
5. [https://supermemo.guru/wiki/Algorithm_SM-17](https://supermemo.guru/wiki/Algorithm_SM-17)
6. [https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VAGUL0)
7. [https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME)
8. [https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/](https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/)


___
### [←Return to homepage](https://expertium.github.io/)
