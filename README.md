I write articles about spaced repetition and especially about [FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS), a modern spaced repetition algorithm used in [Anki](https://apps.ankiweb.net/) and other apps.

The articles are sorted from most important to least important based on 3 factors:

1. Practicality: can this information help you to make any kind of decision? For example, what settings to use in Anki.
2. Uniqueness: are you likely to find this information anywhere else?
3. Amount of effort I have put into writing the article.

About Anki, FSRS and spaced repetition:

1. (unfinished) Benchmark of spaced repetition algorithms. In this article, I analyze results from the [open spaced repetition benchmark repository](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#result), where many modern spaced repetition algorithms are compared based on their ability to predict the probability of recall. This is the most high-effort piece I've ever written. It will be finished when ~~the next major release of Anki comes out~~ ~~when [Jarrett](https://github.com/L-M-Sherlock) finishes modifying the code for the new dataset~~ idk when.
2. [Note types to avoid pattern matching](/Avoid_Pattern_Matching.md). In this article, I go over some note types that can help you avoid the "I memorized what the answer *looks* like rather than the actual information" phenomenon as well as allow you to memorize the same amount of information with fewer cards.
3. [A technical explanation of FSRS](/Algorithm.md), where I explain the formulas that are used to calculate the probability of recall.
4. [Understanding what retention means in FSRS](/Retention.md). I recommend reading this article if you are confused by terms like "desired retention", "true retention" and "average retrievability", the latter two can be found in Stats. True retention table is available in Anki natively since Anki 24.11.
5. [Let's get learning steps wrong!](/Learning_Steps.md) It's much easier than you think.
6. [A short analysis of how review time is related to answer buttons](/Buttons.md). Some wonder, "Do people tend to think longer before pressing Again than before pressing Good?". In this article, I answer that question. Spoiler: yes.<br/> You won't find this information anywhere else; I'm 100% sure that I am the only person on the planet who has analyzed the [Anki 20k dataset](https://huggingface.co/datasets/open-spaced-repetition/FSRS-Anki-20k) to figure out how button usage of Anki users is related to their review times.
7. [Abridged history of spaced repetition](/History.md).
8. [Should you lower your maximum interval?](/Max_Interval.md)
9. [20 reasons why Anki isn't popular](/20reasons.md).
10. [Mass adoption and the perfect UI](/Perfect_UI.md). Can FSRS be easy to use? Yes. But first, we'll need to change ~~some~~ a lot of things.

About other things:

1. [Large Language Models Understand Causality](/LLMCausality.md). Do you think that LLMs are jsut "stochastic parrots"? Then this article is for you!
2. [At what age do great scientists make great discoveries?](/Scientists_Age.md) Is there a "magical" age that is perfect for making groundbreaking discoveries and inventions? Let's investigate it.
3. [Traveling Salesman Problem: an example where "Nearest Neighbor" results in the WORST possible route](/TSP_NN_worst.md). Is it possible to always go  to the nearest point in space, yet end up taking the longest possible path? Yes!

A list of all implementations of FSRS in different programming languages, as well as apps that use FSRS (it's not just Anki!): [https://github.com/open-spaced-repetition/awesome-fsrs.](https://github.com/open-spaced-repetition/awesome-fsrs)

Nothing that floats your boat? [Take a look at these articles and blogs then](/Resources_Dump.md).
