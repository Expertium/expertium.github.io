# Large Language Models Understand Causality

"*LLMs don't understand anything, they just regurgitate what they have seen in the training data without understanding the Deep Connections between Things*" is a very common misconception. And there is very neat paper that tears it to shreds: [https://arxiv.org/pdf/2305.00050](https://arxiv.org/pdf/2305.00050)

It's surprisingly easy to test, and the paper tests it in 2 different ways. Let's start with the first one.

Researchers took examples that look like this:

x: altitude <br />
y: temperature (average over 1961-1990)

x: age in years <br />
y: number of heart beats per minute

x: coarse aggregate -- kg in a m3 mixture <br />
y: concrete compressive strength -- MPa 

x: CO2 emissions for different countries in different years <br />
y: energy use (kg of oil equivalent per capita) for different countries in different years <br />


And so on. Note that each example also has a brief description of the variables, I just omitted it.

Then for each example researchers asked LLMs two questions:

1) Does changing {x} cause a change in {y}?

2) Does changing {y} cause a change in {x}?


Then researchers calculated accuracy across all pairs. **GPT-4 achieved 96% accuracy**.

![image](https://github.com/user-attachments/assets/ceaf5e49-930e-4cea-8147-aa71b56598ad)

Researchers thought "Hmmm, maybe LLMs just saw these pairs in the training data? Let's make new pairs!". GPTs performed well on those as well:

> On this dataset, we applied the exact same (single) prompt from the original benchmark. GPT-3.5-turbo and GPT-4
obtain 80.3% and 98.5% accuracy respectively, indicating that the capability to identify causal direction generalizes to
variable pairs outside of popular datasets.

The second part consisted of fictional toy scenarios like this:

"Alice and Bob each fire a bullet at a window, simultaneously striking the window, shattering it."

Then LLMs were asked two questions:

1) Is {Actor} a necessary cause of {Event}?

2) Is {Actor} a sufficient cause of {Event}?

In the example above the answers are "No" and "Yes", respectively. Alice shooting is not necessary: even if Alice had not fired her bullet at all, Bob's bullet would have struck the window simultaneously and shattered it just the same. Alice shooting is sufficient: her action alone would have been enough to cause the window to shatter.

**GPT-4 achieved 86.6% accuracy** (same across both "necessary" and "sufficient" questions).

Then researchers once again thought "Hmmm, maybe LLMs just saw these questions in the training data? Let's make new pairs!".

**GPT-4 achieved 92.8% on "necessary" questions and 78.5% on "sufficient" questions** made specifically for this paper.


Note that this isn't a recent paper, it's from 2023. And these days there are much smarter models that use Chain-of-Thought reasoning. Of course, LLMs still fail in silly (or deeply inhuman) ways sometimes, but to say that they have no understading of how the world works would be nonsense.


Hopefully my article can convince at least one or two people on this planet to stop saying "stochastic parrots" and "LLMs just regurgitate words with zero understanding".


___
### [←Return to homepage](https://expertium.github.io/)
