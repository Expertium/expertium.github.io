# Benchmark of Spaced Repetition Algorithms

## Intro
This is an extended version of my Reddit post. This article should take approximately 25–30 minutes to read, the Reddit post should take around 11–13 minutes. I tried to make this article suited both to tech-savvy and casual readers, so I may have over-explained some things.
Side note: when I say "we", I'm referring to myself and [LMSherlock](https://github.com/L-M-Sherlock) (Jarrett Ye), the creator of [FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS).

## Metrics
First of all, every spaced repetition algorithm must be able to predict the **probability of recalling** a card at a given point in time, given the card's review history. Let's call that **R**. All three metrics that we use are related to the probability of recall.

If an algorithm doesn't calculate probabilities and just outputs an interval, it's still possible to convert that interval into a probability under certain assumptions. It's better than nothing, since it allows us to perform at least some sort of comparison. That's what we did for SM-2, the only algorithm in the entire benchmark that wasn't originally designed to predict probabilities. We decided not to include [Memrise](https://memrise.zendesk.com/hc/en-us/articles/360015889057-How-does-the-spaced-repetition-system-work) because we are unsure if the assumptions required to convert its intervals to probabilities hold. Well, it wouldn't perform great anyway, it's about as inflexible as possible.

Once we have an algorithm that predicts R, we can run it on some users' review histories to see how much predicted R deviates from measured R. If we do that using hundreds of millions of reviews, we will get a very good idea of which algorithm performs better on average. **RMSE**, or root mean square error, can be interpreted as "the average difference between predicted and measured probability of recall (R)". RMSE is a measure of **calibration**.

Loosely speaking, if an algorithm predicts an X% chance of something happening, it should happen X% of the time. For example, if a weatherman says "There is a 90% chance of rain today", it should rain on 90% of days when he said that. That's good calibration. If it rained only on 40% of those days, it means that the weatherman (or, well, his forecasting system) is poorly calibrated - his probabilities don't match observed frequencies. RMSE measures how well-calibrated an algorithm is.

The calculation of RMSE has been reworked in the past to prevent cheating, aka algorithms achieving good numbers on paper without getting better at predicting R in reality. If you want to know the nitty-gritty mathematical details, you can read this article by LMSherlock and me. The new method is our own invention, and you won't find it in any academic paper. Anki >=24.04 use the new method when "Evaluate" is used.

Here's what you need to know about RMSE:
1. RMSE measures how close the predicted R is to reality. It puts reviews into bins and measures the difference between *average* actual retention and *average* predicted R within each bin.
2. RMSE ranges from 0 to 1, lower is better.
3. We calculate it by binning reviews in a way that is specific to spaced repetition, you won't find this in the literature. This isn't very nice for making the benchmark results accepted by other researchers, but we have other metrics for that.
4. It's not what the optimizer in Anki is minimizing. Log loss is what the FSRS optimizer is internally used for optimization, not RMSE. We can't use RMSE for that.

Next is log loss. Here's what you need to know about log loss:
1. Log loss measures how close the predicted probability of recall (R) is to reality, just like RMSE. However, unlike RMSE, it doesn't rely on binning reviews. RMSE is based on the difference between averages, whereas log loss is based on the difference between individual predictions and individual review outcomes.
2. Log loss ranges from 0 to infinity, lower is better.
3. Unlike RMSE, log loss never reaches 0, unless the algorithm only outputs ones and zeros. If an algorithm outputs numbers between 0 and 1, the minimum possible value of log loss that it can achieve is >0. This makes log loss less intuitive than RMSE. You might intuitively expect that if the distribution of predicted probabilities exactly matches the distribution of true probabilities, the loss would be zero, but no.

Next is AUC. Unlike the previous two metrics, AUC is not a measure of **calibration** but of **discrimination**. Here's what you need to know about AUC:
1. AUC measures how well an algorithm can tell classes apart; in our case, classes are "recalled" and "forgotten." You can think of AUC as a measure of how well the algorithm can draw a boundary between two classes, such that all members of class 1 are on one side of the boundary, and all members of class 2 are on the other side.
2. AUC scores range from 0 to 1, however, in practice it's almost always greater than 0.5. AUC scores less than 0.5 indicate that the algorithm is performing worse than random. Higher is better.

AUC can be rather unintuitive in some cases. Exaggerated example: suppose you have an algorithm always outputs 99% probability of having cancer for people who do have cancer, and always outputs a 98% probability of having cancer for people who do not have cancer. What do you think is the AUC score of this algorithm? Answer: 1.0, because it can perfectly distinguish between these two classes, even if the calibration is absolutely terrible. AUC doesn't tell us anything about calibration, only about discrimination. As long as the algorithm always outputs values higher than some threshold for class 1 and lower than that threshold for class 2, it will have an AUC score of 1.

Below is a table comparing different metrics.

![Comparison table (metrics)](https://github.com/user-attachments/assets/42aeaf91-0a5b-43bf-92e6-23f1c88c42c7)


## Algorithms

### FSRS family

​1.​ ​FSRS v3. It was the first version of FSRS that people actually used, it was released in October 2022. It wasn't terrible, but it had issues. LMSherlock, I, and several other users have proposed and tested several dozens of ideas (only a handful of them proved to be effective) to improve the algorithm.
​
2​. ​FSRS v4. It came out in July 2023, and at the beginning of November 2023, it was integrated into Anki. It's a significant improvement over v3.

​3​. ​FSRS-4.5. It's a slightly improved version of FSRS v4, the shape of the forgetting curve has been changed.

4. FSRS-5. The newest version. The main difference is that it takes into account same-day reviews, unlike all previous version, though the improvement in performance is small. Anki 24.XX uses it, AnkiMobile and AnkiDroid haven't been updated yet, so they use FSRS-4.5. If you want to read about the differences between FSRS-4.5 and FSRS-5, I have an article with an in-depth explanation of FSRS formulas.

5. FSRS-5 (default parameters). This is just to see how well FSRS-5 performs without optimization.

6. FSRS-5 (pretrain). In FSRS, the first 4 parameters (values of initial stability) are optimized in a completely different way, compared to the rest. "Pretrain" is when the first 4 parameters are optimized, while the rest of parameters are set to default. In Anki >=24.06, when parameters are optimized, the optimizer determines whether to keep the default parameters, perform pretrain, or perform a full optimization; which one is used depends on the user's review history. The more reviews a user has, the more likely it is that full optimization will be performed.

Below is a diagram that should give you a better understanding of FSRS.

![FSRS](https://github.com/user-attachments/assets/89dd0ca5-6579-4471-bf95-2b1c3535f881)

In order to calculate the length of the next interval, FSRS requires the length of the previous interval, grade (Again/Hard/Good/Easy) and its own previous state, which is represented using three numbers: Difficulty, memory Stability, and Retrievability (DSR). Notice that horizontal arrows always point to the right, showing that past states can affect future states, but future states cannot affect past states.


### General-purpose machine learning algorithms family

7. Transformer. This neural network architecture has become popular in recent years because of its superior performance in natural language processing. ChatGPT uses this architecture. In our implementation, Transformer calculates memory stability as an intermediate value and employs an exponential forgetting curve.

8. GRU, Gated Recurrent Unit. This neural network architecture is commonly used for time series analysis, such as predicting stock market trends or recognizing human speech. Originally, we used a more complex architecture called LSTM, but GRU performed better with fewer parameters.

![GRU](https://github.com/user-attachments/assets/49f3152b-524f-46d3-b202-4b0090f921d0)

GRU is also a recurrent algorithm, just like FSRS, even if the mathematical formulas are completely different. Its state is represented using one number. GRU also calculates memory stability as an intermediate value and employs an exponential forgetting curve.

9. GRU-P. Unlike GRU, which predicts memory stability before converting it into R via an exponential forgetting curve formula, GRU-P predicts R directly. More about GRU-P later.

10. GRU-P (short-term). Same as above, but it also uses same-day reviews, so it's trained on more data.


### DASH family

11. [DASH](https://scholar.colorado.edu/concern/graduate_thesis_or_dissertations/zp38wc97m), Difficulty, Ability and Study History. This is an actual *bona fide* model of human memory based on neuroscience. Well, kind of. The issue with it is that the forgetting curve looks like a step function.

12. DASH[MCM]. A hybrid model, it addresses some of the issues with DASH's forgetting curve.

13. DASH[ACT-R]. Another hybrid model, it finally achieves a smooth forgetting curve.

[Here](https://www.politesi.polimi.it/retrieve/b39227dd-0963-40f2-a44b-624f205cb224/2022_4_Randazzo_01.pdf) is another relevant paper.

![DASH](https://github.com/user-attachments/assets/b0b0b3c7-998e-4c77-8955-f9cb650bc180)

DASH, DASH[MCM] and DASH[ACT-R] don't have state variables that are carried on between reviews, and they don't process reviews sequentially, like SM-17/18 or FSRS. They are more like Transformers: all past reviews must be processed in order to calculate the length of the next interval. This makes them much slower than FSRS when the number of reviews is large. In FSRS, each review takes a constant amount of time to process. If a card has 100 reviews, processing the first review will take the same amount of time as processing the 100th review. In DASH, the processing time of a single review depends on the number of past reviews. Therefore, processing the 100th review takes much longer than processing the first one.

Also, even though the diagram shows "Next interval length" as output, in reality, calculating the next interval using DASH algorithms would be very difficult due to the quirks of their forgetting curves. With FSRS and HLR, calculating the probability of recall that corresponds to a specific interval length and calculating the interval length that corresponds to a specific probability of recall is equally easy, but not with DASH algorithms. Still, I drew the diagram this way because a diagram that shows how algorithms predict probabilities would be much harder to read.


### Other algorithms

14. [ACT-R](http://act-r.psy.cmu.edu/wordpress/wp-content/themes/ACT-R/workshops/2003/proceedings/46.pdf), Adaptive Control of Thought - Rational (I've also seen "Character" instead of "Control" in some papers). It's a model of human memory that makes one very strange assumption: whether you have successfully recalled your material or not doesn't affect the magnitude of the spacing effect, only the interval length matters. Simply put, this algorithm doesn't differentiate between Again/Hard/Good/Easy.

![ACT-R](https://github.com/user-attachments/assets/f566d8d2-9d33-4f19-868f-313b9dfd2866)

Below are some example forgetting curves.

![Forgetting curves 2](https://github.com/user-attachments/assets/2a3baf64-f3bc-41cc-acdd-57dd31a0b213)

These curves were plotted using default parameters, which have been obtained by running each algorithm on 20 thousand collections of Anki users. So what you're seeing are "average" or "typical" curves.
DASH's curve looks like a step function, which goes against our human intuition and common sense. DASH[MCM] attempts to smooth it, but you can see that it's not perfect. DASH[ACT-R] achieves a smooth curve. In ACT-R and DASH, probability of recall doesn't start from 100%.

15. [HLR](https://github.com/duolingo/halflife-regression/blob/master/settles.acl16.pdf), Half-Life Regression. It's an algorithm developed by Duolingo for Duolingo. The memory half-life in HLR is conceptually very similar to the memory stability in FSRS, but it's calculated using an overly simplistic formula.

![HLR](https://github.com/user-attachments/assets/5bfd0b0a-d030-4889-a478-27a8f520cbd9)

For HLR, the order of reviews doesn't matter because it only requires summary statistics about the whole review history. Regardless of how you rearrange reviews, the total number of reviews, passed reviews, and failed reviews (lapses) will remain the same.

16. SM-2. It's a 35+ year old algorithm that is still used by Anki, Mnemosyne, and possibly other apps as well. It's main advantage is simplicity. Note that in our benchmark it is implemented the way it was originally designed. It's not the Anki version of SM-2, it's the original SM-2. We put a not-so-rigorous interval-to-probability converter on top of it.

17. NN-17. It's a neural network approximation of [SM-17](https://supermemo.guru/wiki/Algorithm_SM-17). The SuperMemo wiki page about SM-17 may appear very detailed at first, but it actually obfuscates all of the important details that are necessary to implement SM-17. It tells you what the algorithm is doing, but not how. Our approximation relies on the limited information available on the formulas of SM-17, while utilizing neural networks to fill in any gaps.

![NN-17](https://github.com/user-attachments/assets/f877e8f9-f06c-46c7-9573-335bfccb196b)

In order to calculate the length of the next interval, NN-17 requires the length of the previous interval, grade (Again/Hard/Good/Easy) and its own previous state, which is represented using four numbers: Difficulty, memory Stability, Retrievability and the number of lapses.

18. AVG. It's an "algorithm" that outputs a constant equal to the user's average retention. For example, if the user presses Hard/Good/Easy 85% of the time, the "algorithm" will always output an 85% probability of recall for any given review. You can think of it as a weatherman who says "The temperature today will be average, the wind speed will be average, and the humidity will be average as well" every single day. This "algorithm" is intended only to serve as a baseline for comparison and has no practical applications.

I did my best to create a "taxonomy" of spaced repetition algorithms.

![All algorithms (new) 1 1](https://github.com/user-attachments/assets/f676cef3-0807-4ec3-a0f4-30c91006677b)

SM-2 is not included in this diagram because it wasn't designed to predict the probability of recall, unlike the other algorithms.
SM-17/18 algorithms also use the three-component (Difficulty, Stability, Retrievability) model of memory.
HLR lacks a difficulty variable, but it does have memory stability (half-life) and retrievability, so one could say that it employs a two-component model of memory, rather than a three-component model.

Additionally, below is a table comparing different algorithms.

![Comparison table](https://github.com/user-attachments/assets/642fb300-e43e-4a84-a85d-06fb884fe679)







