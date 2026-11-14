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

Let's start with the forgetting curve. FSRS v3 used an exponential function. In FSRS v4, it was replaced by a power function that provided a better fit to the data. FSRS-4.5 introduced a different power function that provided an even better fit, and FSRS-5 used that curve as well. FSRS-6 added an optimizable parameter (w20), which lets the curve adapt to each user. Finally, FSRS-7 uses a weighted average of two power forgetting curves.

Below, R denotes the probability of recall, S denotes memory stability, D denotes difficulty, and t denotes the time since the last review.

<img width="2440" height="3062" alt="fig_forgetting_curves" src="https://github.com/user-attachments/assets/69d8624b-c46c-4957-8ddc-2049d1cf2321" />

In total, FSRS-7's curve has 11 parameters. While it may seem like it will overfit and fail to generalize, it has lower log loss in the [srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark), which uses time series splits. Simply put, this formula works better on data that FSRS wasn't trained on, compared to FSRS-6's one parameter formula.

Btw, I use 0-based notation for the parameters of all versions: "zero, one, two", not "one, two, three".

<img width="482" height="414" alt="image" src="https://github.com/user-attachments/assets/9b14d5ee-065a-4c88-b8b8-8904406883a9" />

Below is a table that compares the curve shapes of different FSRS versions. As FSRS evolved, its curve shape was changed to better fit [the data from the 10k dataset](https://huggingface.co/datasets/open-spaced-repetition/anki-revlogs-10k). FSRS-6 made the curve shape adaptive, so it differs between users, and FSRS-7 made it even more adaptive. I would even go as far as to say that increasing the expressiveness of FSRS formulas (not just the forgetting curve) was the main driving force behind all FSRS version upgrades.

<img width="2964" height="908" alt="fig_curve_shape_table" src="https://github.com/user-attachments/assets/78a513c4-81bf-40ea-91ba-a79be7dd3580" />

All versions before FSRS-7 used the three-component model of memory: Difficulty, Stability, Retrievability. FSRS-7 uses four: stability is split into long-term stability (S_long) and short-term stability (S_short).
Unlike previous versions, where the interval at a given level of desired retention could be calculated analytically (aka using a single formula), this is no longer possible in FSRS-7, so intervals are calculated iteratively.
Also, in previous versions memory stability—amount of time needed for R to drop from 100% to 90%, I'll call it S90—was both a number used internally and a number the user could see. In FSRS-7 S90 is not used internally, inside FSRS-7, but it can still be calculated separately; for example, to be shown in Anki's Stats window.

FSRS-7's formula is quite complex, so let me try to explain what each part does.

R is the final output. It's a weighted average of R_short and R_long. The weights *themselves* depend on S_short and S_long. This means that the shape of the forgetting curve can vary not only between users, and not even only between cards, but **even change over the course of a card's lifetime**. Also, the shape of the curve depends on Difficulty now, see W_long and R_long.

d_short and f_short control the shape of the short-term forgetting curve. W_short controls whether the final curve is more similar to the short-term curve or to the long-term curve. A greater W_short means that the final curve is more similar to the short-term curve. Note that the short-term curve's shape and weight don't depend on Difficulty.
d_long and f_long control the shape of the long-term forgetting curve. A greater W_long means that the final curve is more similar to the long-term curve. W_long depends on D. R_long also has another factor, e^((w_32 - 0.3)(D-5)). You can think of that factor this way: for difficult cards, it's *as if* time passes faster. Every minute, 3 minutes pass for a difficult card. That may sound silly, but I can't think of a better way to explain it. The long-term curve's shape and weight both depend on D.

Also, unlike all previous versions that were designed to work with integer interval lengths (0 days, 1 day, 2 days, etc.), FSRS-7 is designed to work with fractional interval lengths. So **FSRS-7 can finally properly model the forgetting curve during same-day reviews**, which no previous FSRS version was good at.

You don't need to understand the math (unless you are a researcher). The simple version is: **the shape of the curve depends on each card's history and can change after every review. It is affected by the two stability values and by Difficulty.**

Here is a comparison of all forgetting curves with default parameters of each FSRS version. S=10 days for all versions except for FSRS-7.

<img width="1800" height="1040" alt="fig_r_vs_t" src="https://github.com/user-attachments/assets/33b22cc4-d591-448f-a775-b118e8907f6b" />

Notice that FSRS-7's curve falls off very steeply at first, but is flatter than any other curve after that. This specific shape—steep fall-off initially, then flatter later—allowed FSRS-7 to become far more accurate on same-day reviews than FSRS-6 (and modestly more accurate on long-term reviews as well).

## S, Stability

FSRS-7 has two values of memory stability, S_long and S_short. Neither can be interpreted as "how long it takes for R to drop from 100% to 90%". They do not have any intuitive interpretation, but, as I said above, you can still calculate the intuitive S90 separately.

Here are the formulas for S_long and S_short. The formula is the same, but it uses different parameters and retrievability values depending on which S is being computed. The S_long update uses R (weighted average of two curves), while the S_short update uses R_short.

<img width="1195" height="1013" alt="fig_s_success" src="https://github.com/user-attachments/assets/22b33ace-0b4f-4f43-865c-ed245ddb3b5d" />

The new value of memory stability is equal to the previous value multiplied by a factor that we will call S_Inc (Inc stands for "increase"). S_Inc >= 1. In other words, **memory stability cannot decrease if the review was successful**. Easy, Good, and Hard all count as "success"; Again counts as a memory "lapse". That's why you shouldn't use Hard as a failing grade, only as a passing grade.

**Important takeaway number one: the larger the value of D, the smaller the SInc value. This means that the increase in memory stability for difficult material is smaller than for easy material.**

**Important takeaway number two: the larger the value of S, the smaller the SInc value. This means that the higher the stability of the memory, the harder it becomes to make the memory even more stable. Memory stability tends to saturate.**

**Important takeaway number three: the smaller the value of R, the larger the SInc value. This means that the best time to review your material is when you almost forgot it (provided that you succeeded in recalling it).**
If that sounds counter-intuitive, imagine if the opposite was true. Imagine that the best time to review your material is when you know it perfectly (R is very close to 100%). There is no reason to review something if you know it perfectly, so this can't be true.

The formula for the next value of S is different if the user pressed "Again".

<img width="1134" height="735" alt="fig_s_lapse" src="https://github.com/user-attachments/assets/e2ef30d5-ef92-45a9-88ef-3850c8f26275" />

min(S, S_fail) is necessary to ensure that post-lapse stability can never be greater than stability before the lapse. Interesting fact: in FSRS-6 there was a D-related term, but it turns out that this formula works just fine even without D, so in FSRS-7 it no longer depends on D.

Since both formulas require the previous value of S to calculate the next value, they cannot be used to estimate initial stability on the first review; there is no such thing as a "zeroth review". So the initial stability values for the four answer buttons (Again, Hard, Good, and Easy) are learnable parameters. FSRS-6 estimated them separately from the main optimization. In FSRS-7 all parameters are optimized jointly, without using different methods for different parameters.

<img width="1147" height="678" alt="fig_s_initial" src="https://github.com/user-attachments/assets/6b5ac1de-e72c-414c-967a-96ea35c545c4" />

## D, Difficulty

Initial D, after the first review:

<img width="1381" height="319" alt="fig_d_initial" src="https://github.com/user-attachments/assets/a894c64e-0c00-49ee-bcaf-d5afdfbd9aac" />

Clamp means that if D is less than 1 then it becomes 1, and if D is more than 10 then it becomes 10. In other words, D is bounded between 1 and 10. Note that D doesn't have a precise definition; it's a heuristic.

D formula for all other reviews:

<img width="1374" height="733" alt="fig_d_updates" src="https://github.com/user-attachments/assets/0626efd9-53c4-4a6b-8052-fad317a73f1a" />

(10-D)/9 has a damping effect: as D approaches 10, updates shrink so that D is never exactly 10, only *approaching* 10. Additionally, D is very slightly pulled toward the initial D for Easy. 0.01 and 0.99 were previously one optimizable parameter, but it turned out that using constants doesn't affect accuracy at all, so that parameter was removed.

L is a new term that makes D depend on R, **but only when you press Again, not Hard, Good, or Easy**. In other words, **it makes difficulty updates depend on how likely you were to recall the card if you ended up forgetting it (pressing Again).** However, it only improves accuracy very mildly, and if someone wanted to simplify FSRS-7, this could easily be cut out. While our intuition suggests that D updates should vary greatly depending on whether you were on the brink of forgetting the card, it turns out that this has little effect.

Note: D ranges from 1 to 10, but in Anki D is displayed as a number between 0 and 1. This conversion is completely unnecessary in my opinion.

## Optimization

You can find the training pipeline and hyperparameters in [srs-benchmark](https://github.com/open-spaced-repetition/srs-benchmark) (Python) and in [fsrs-rs](https://github.com/open-spaced-repetition/fsrs-rs) (Rust).

## LLM usage disclosure

FSRS-7's formulas were developed mostly by humans ([1DWalker](https://github.com/1DWalker) and [Expertium](https://github.com/Expertium)), with some help from LLMs. See the [FSRS autoresearch repository](https://github.com/Expertium/fsrs-autoresearch). For example, the forgetting curve originally had 8 parameters, not 11. And originally I wasn't very keen on the idea of splitting S into S_long and S_short, but the results were too good to pass up.

The optimizer that is used for FSRS-6 and FSRS-7 was greatly sped up entirely by LLMs, with very little human guidance. See the [optimizer speedup project](https://github.com/Expertium/fsrs-rs-speed-autoresearch). FSRS-6 parameters are now optimized faster, and FSRS-7's optimizer will be even faster.

## Post-FSRS future

FSRS-7 is most likely the last version of FSRS. There's simply nothing left to squeeze in terms of accuracy (log loss, more precisely, I just use "accuracy" colloquially since not everyone knows what log loss is). All low-hanging fruit has been picked up, and not-so-low-hanging fruit as well. However, FSRS-7 still has weaknesses:

1) It only uses interval lengths and grades. It doesn't know whether you reviewed this card after doing 5 other reviews or after 500. It doesn't know whether you reviewed it at 7 AM on Tuesday or at 9 PM on Sunday.

2) It treats all cards as independent, even cards that came from the same day and *even siblings* (sibling dispersion in Anki is not part of the FSRS algorithm, it's a separate thing, like a cherry on top of a cake).

3) It doesn't take into account card content.

In srs-benchmark, RWKV (a type of neural network) addresses issues 1 and 2, though it's still in development and [JSchoreels](https://github.com/JSchoreels) (Sound in the Anki Discord server) is releasing new versions regularly. RWKV will most likely never be implemented in Anki, but there are forks that use it (and FSRS-7 too). See:

Desktop: [JSchoreels Anki fork releases](https://github.com/JSchoreels/anki/releases). I'm planning to make my own fork too, but sadly, I'll have to postpone working on it until mid- to late 2027.

AnkiDroid: [Kuma3 AnkiDroid fork releases](https://github.com/Pedrubik2000/kuma3-anki-app/releases).

So if you want to try out FSRS-7 and RWKV while Anki still uses FSRS-6, you can try these forks.
As for issue 3, we, the FSRS team, do not have a dataset with card content. AnkiHub does, but it doesn't seem like they will share it with us.

___
### [←Return to homepage](https://expertium.github.io/)
