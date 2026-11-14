# A technical explanation of FSRS-7

If you want a simple overview first, start here: [ABC of FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS).

In this article, I'll explain how FSRS-7 works. Note that FSRS-7 is not yet available in Anki as of November 2026.

# Table of contents
- [R, Retrievability](#r-retrievability)
- [S, Stability](#s-stability)
- [D, Difficulty](#d-difficulty)
- [Optimization](#optimization)
- [LLM usage disclosure](#llm-usage-disclosure)
- [Post-FSRS future](#post-fsrs-future)

## R, Retrievability

Let's start with the forgetting curve. In FSRS v3, an exponential function was used. In FSRS v4, the exponential function was replaced by a power function, which provided a better fit to the data. Then, in FSRS-4.5, it was replaced by a different power function which provided an even better fit. It was used in FSRS-5 as well. Then FSRS-6 added an optimizable parameter (w20) to the curve to make it personalizable for every user.
Finally, FSRS-7 uses a weighted average of two power forgetting curves.

<img width="2440" height="3062" alt="fig_forgetting_curves" src="https://github.com/user-attachments/assets/69d8624b-c46c-4957-8ddc-2049d1cf2321" />

In total, FSRS-7's curve has 11 parameters. While it may seem like it will overfit and fail to generalize, it has lower log loss in the [srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark), which uses time series splits. Simply put, this formula works better on data that FSRS wasn't trained on, compared to FSRS-6's one parameter formula.

Note that for all parameters of all versions I use 0-based notation. Not "one, two, three", but "zero, one, two".

<img width="482" height="414" alt="image" src="https://github.com/user-attachments/assets/9b14d5ee-065a-4c88-b8b8-8904406883a9" />

Here is a comparison of all forgetting curves with default parameters of each FSRS version:

<img width="1800" height="1040" alt="fig_r_vs_t" src="https://github.com/user-attachments/assets/33b22cc4-d591-448f-a775-b118e8907f6b" />

Notice that FSRS-7's curve falls off very steeply at first, but is flatter than any other curve after that. This specific shape - steep fall off initially, flat later - is what allowed FSRS-7 to become far more accurate on same-day reviews than FSRS-6 (and modestly more accurate on long-term reviews as well).

All versions before FSRS-7 used the three-component model of memory: Difficulty, Stability, Retrievability. FSRS-7 uses four: stability is split into long-term stability (S_long) and short-term stability (S_short).
Unlike previous versions, where the interval at a given level of desired retention could be calculated analytically (aka using a single formula), this is no longer possible in FSRS-7, so intervals are calculated iteratively.
Also, in previous versions memory stability - amount of time needed for R to drop from 100% to 90%, I'll call it S90 - was both a number used internally and a number the user could see. In FSRS-7 S90 is not used internally, inside FSRS-7, but it can still be calculated separately; for example, to be shown in Anki's Stats window.

FSRS-7's formula is quite complex, so let me try to explain what each part does.

R is the final output. It's a weighted average of R_short and R_long. The weights *themselves* depend on S_short and S_long. This means that the shape of the forgetting curve can vary not only between users, and not even only between cards, but **even change over the course of a card's lifetime**. Also, the shape of the curve depends on Difficulty now, see W_long and R_long.

d_short and f_short control the shape of the short-term forgetting curve. W_short controls whether the final curve is more similar to the short-term curve or to the long-term curve. Greater W_short = final curve is more similar to the short-term curve. Note that short-term curve's shape and weight don't depend on Difficulty,.
d_long and f_long control the shape of the long-term forgetting curve. Greater W_long = final curve is more similar to the long-term curve. W_long depends on D. R_long also has another factor, e^((w_32 - 0.3)(D-5)). You can think of that factor this way: for difficult cards it's *as if* time passes faster. Every minute, 3 minutes pass for a difficult card. That sounds kinda silly when I put this way, but I can't think of a better way to put it. Long-term curve's shape and weight both depend on D.

You don't need to understand the math (unless you are a researcher). The simple version is: **the shape of the curve now depends on each card's history and can change after every review, two Stabilities and Difficulty affect it**.



Also, unlike all previous versions that were designed to work with integer interval lengths (0 days, 1 day, 2 days, etc.), FSRS-7 is designed to work with fractional interval lengths. So **FSRS-7 can finally properly model the forgetting curve during same-day reviews**, which no previous FSRS version was good at.

## S, Stability

FSRS-7 has two values of memory stability, S_long and S_short. Neither can be interpreted as "how long it takes for R to drop from 100% to 90%". They do not have any intuitive interpretation, but, as I said above, you can still calculate the intuitive S90 separately.

Here are the formulas for S_long and S_short. They are identical, but use different parameters.

## D, Difficulty

## Optimization

You can find the training pipeline and hyperparameters in [srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark) (Python) and in [fsrs-rs](https://github.com/open-spaced-repetition/fsrs-rs) (Rust).

## LLM usage disclosure

FSRS-7's formulas were developed mostly by humans (http://github.com/1DWalker and https://github.com/Expertium), with some help from LLMs. See https://github.com/Expertium/fsrs-autoresearch.
The optimizer that is used for FSRS-6 and FSRS-7 was greatly sped up entirely by LLMs, with very little human guidance. See https://github.com/Expertium/fsrs-rs-speed-autoresearch.

## Post-FSRS future

FSRS-7 is most likely the last version of FSRS. There's simply nothing left to squeeze. All low-hanging fruit has been picked up, and not-so-low-hanging fruit as well. However, FSRS-7 still has weaknesses:
1) It only uses interval lengths and grades. It doesn't know whether you reviewed this card after doing 5 other reviews or after 500. It doesn't know whether you reviewed it at 7 AM on Tuesday or at 9 PM on Sunday.
2) It treats all cards as independent, even cards that came from the same day and *even siblings* (sibling dispersion in Anki is not part of the FSRS algorithm, it's a separate thing, like a cherry on top of a cake).
3) It doesn't take into account card content.

In srs-benchmark, RWKV (a type of neural network) addresses issues 1 and 2. RWKV will most likely never be implemented in Anki, but there are forks that use it (and FSRS-7 too). See:
Desktop: https://github.com/JSchoreels/anki/releases
AnkiDroid: https://github.com/Pedrubik2000/kuma3-anki-app/releases

So if you want to try out FSRS-7 and RWKV while Anki still uses FSRS-6, you can try these.
As for issue 3: we, FSRS team, do not have a dataset with card content. AnkiHub does, but it doesn't seem like they will share it with us.
