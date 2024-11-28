I write articles about spaced repetition and especially about [FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS), a modern spaced repetition algorithm used in [Anki](https://apps.ankiweb.net/). That's all, end of the intro. Here are the articles, (kinda) sorted from most important to least important:

1. (unfinished)[Benchmark of spaced repetition algorithms](/Benchmark.md). In this article, I analyze results from the [open spaced repetition benchmark repository](https://github.com/open-spaced-repetition/srs-benchmark?tab=readme-ov-file#result), where many modern spaced repetition algorithms are compared based on their ability to predict the probability of recall. This is the most high-effort piece I've ever written. It will be finished when ~~the next major release of Anki comes out~~ when LMSherlock finishes modifying the code for the new dataset.
3. [A technical explanation of FSRS](/Algorithm.md), where I explain the formulas that are used to calculate the probability of recall.
4. [Note types to avoid pattern matching](/Avoid_Pattern_Matching.md). In this article, I go over some note types that can help you avoid the "I memorized what the answer *looks* like rather than the actual information" phenomenon as well as allow you to memorize the same amount of information with fewer cards.
5. [Understanding what retention means in FSRS](/Retention.md). I recommend reading this article if you are confused by terms like "desired retention", "true retention" and "average predicted retention", the latter two can be found in Stats if you have the [FSRS Helper add-on](https://ankiweb.net/shared/info/759844606) installed and press Shift + Left Mouse Click on the Stats button.
6. [A short analysis of how review time is related to answer buttons](/Buttons.md). Some wonder, "Do people tend to think longer before pressing Again than before pressing Good?". In this article, I answer that question. Spoiler: yes.<br/> I rank this as relatively important simply because you won't find this information anywhere else.
7. [Abridged history of spaced repetition](/History.md).
8. [Let's get learning steps wrong!](/Learning_Steps.md) It's much easier than you think.
9. [Should you lower your maximum interval?](/Max_Interval.md) 
10. [Mass adoption and the perfect UI](/Perfect_UI.md). Can FSRS be easy to use? Yes. But first, we'll need to change ~~some~~ a lot of things.

A list of all implementations of FSRS in different programming languages, as well as apps that use FSRS (it's not just Anki!): [https://github.com/open-spaced-repetition/awesome-fsrs.](https://github.com/open-spaced-repetition/awesome-fsrs)

Nothing that floats your boat? [Take a look at these articles and blogs then](/Resources_Dump.md).
