# A technical explanation of FSRS-7

If you want a simple overview first, start here: [ABC of FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS).

In this article, I'll explain how FSRS-7 works. Note that FSRS-7 is not yet available in Anki as of November 2026.

# Table of contents
- [R, Retrievability](#r-retrievability)
- - [Intervals](#intervals)
- [S, Stability](#s-stability)
- [D, Difficulty](#d-difficulty)
- [Optimization](#optimization)
- [LLM usage disclosure](#llm-usage-disclosure)

## R, Retrievability

Let's start with the forgetting curve. In FSRS v3, an exponential function was used. In FSRS v4, the exponential function was replaced by a power function, which provided a better fit to the data. Then, in FSRS-4.5, it was replaced by a different power function which provided an even better fit. It was used in FSRS-5 as well. Then FSRS-6 added an optimizable parameter (w20) to the curve to make it personalizable for every user.
Finally, FSRS-7 uses a weighted average of two power forgetting curves.



