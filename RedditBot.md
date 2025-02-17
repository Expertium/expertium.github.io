# Reddit Bot

I made [u/FSRS_bot](https://www.reddit.com/user/FSRS_bot/) on Reddit, it helps people on [r/Anki](https://www.reddit.com/r/Anki) and [r/medicalschoolanki](https://www.reddit.com/r/medicalschoolanki) with FSRS-related questions.
In this article, I want to talk about all the different things I did to improve it. This isn't super educational and quite technical, though you will learn a thing or two about machine learning.
I won't be showing my entire code since it's too spaghetti, but I will show some code snippets. If you ever think that my storytelling sounds like a mess, just keep in mind that the reality was even messier and I did 10x more dumb stuff than this article mentions, and everything took 10x longer than you might expect.

Also, this is meant to be more like a diary of my journey rather than a guide/textbook for other people, so don't be surprised if this article feels like I was simultaneously playing 5 games of chess and overdosing on adderall while writing it.

## Part One: Scraping

So first I needed data aka Reddit posts and comments. I used [PRAW](https://praw.readthedocs.io/en/stable/) for that. A long time ago I used it to make a notifier to respond to posts myself, but quickly realized that it's exhausting.
I have changed it several times, and I wasn't keeping track of how many posts I had at any given moment, so I will only give the final number (as of 20.11.2024): **1,427 posts and 92 comments, 1,519 training examples in total.** Most of them are from r/Anki, some from r/medicalschoolanki, and a handful of them are from r/AnkiMCAT and a few other subreddits. While the initial plan was to keep only FSRS-related posts, later I added a bunch of other posts for the sake of training my language model on diverse data.

The scraping code looks kinda like this:

```python
reddit = praw.Reddit(username="MyUsername", password="123", client_id="123", client_secret="123", user_agent="praw_scraper")
subreddit = reddit.subreddit('Anki')
post_ids = []
post_text = []
# sort by new
for post in subreddit.new(limit=9999):
    for keyword in all_keywords:
        text = str(post.title) + ' ' + str(post.selftext)
        if keyword in text and post.id not in post_ids:  # check that the post is relevant and that we haven't scraped it before
            post_ids.append(post.id)
            post_text.append(text)
```

This is a simplified version. In reality it has a few more checks and I'm not only sorting by new, I'm also doing this for sort by hot, by rising, by controversial, by top (week, month and year), and do a few searches to find posts that contain, for example, "FSRS". Also, I need to write IDs and text to my disk to store them.

## Part Two: Keyword Matching and the First Classifier

My initial idea was to just check if the post contains "FSRS", or one of its misspellings ("FRS", "FRSRS", "FSES", "FRSR", etc.), and make the bot comment the same generic message. However, I quickly realized that there are different types of questions, and they require personalized answers. So I categorized posts into 22 categories:

1​)​ Desired Retention. A post about choosing the value of desired retention and/or using "Compute minimum recommended retention".

2​)​ Optimization. A post about the optimization of FSRS parameters.

3​)​ Learning Steps. A post about choosing the right learning steps when using FSRS.

4​)​ Interval Length. A post where the person either feels like the intervals are too long or too short (most of the time it's the former).

5​)​ Exam. A post where the person is askign what's the best way to use FSRS before an exam.

6​)​ Helper Add-on. A post about the Helper add-on.

7​)​ Metrics. A post about RMSE and/or log loss values.

8​)​ Easy Days. A post about the Easy Days feature of the add-on. Once Anki .11 will come out, posts about the native Easy Days functionality will also be labeled with this label.

9​)​ Load Balance. A post about the Load Balance feature of the add-on.

10​)​ Disperse Siblings. A post about the Disperse Siblings feature of the add-on.

11​)​ Fuzz. A post about fuzz *and* FSRS, not just fuzz. One of the rarest types of posts.

12​)​ SM-2 Retention. A post about SM-2 retention (renamed to "Historical retention" at some point). One of the rarest types of posts.

13​)​ Reschedule. A post about using the "Reschedule cards on change" option.

14​)​ Platforms. A post where the person is asking whether FSRS is supported in AnkiDroid, AnkiMobile, AnkiWeb; or something along those lines.

15​)​ Should. A post where hte person has doubts about switching to FSRS.

16​)​ AO (automatic optimization). A post about *automatic* optimization of parameters and why it's not implemented yet. One of the rarest types of posts. One of the rarest types of posts.

17​)​ ETK (estimated total knowldge). A post about the new stat: [estimated total knowledge](https://docs.ankiweb.net/stats.html#the-graphs). One of the rarest types of posts.

18​)​ Jarrett. A post that is addressed to LMSherlock (Jarret Ye) or is about some technical stuff related to the FSRS development. One of the rarest types of posts.

19​)​ General. A post where the person is asking what is FSRS, how to configure it, or a whole bunch of things at once.

20​)​ TR<DR. True retention is below desired retention. Well, posts where true retentino is higher than desired are lumped in here anyway. Basically, it's when there is a discrepancy.

21)​ Null. Either unrelated to FSRS or there is no reason to send the bot to reply to this person.

22)​ Simulator. A recently added label, since in Anki 24.11 there is now a new FSRS-related thingy, similar to [this](https://ankiweb.net/shared/info/817108664).

And yes, I read each of the 1,427 posts and 92 comments and labeled all of them manually.

![image](https://github.com/user-attachments/assets/41a9a3cb-b1a2-4ae5-82c6-dde776fe1037)

Btw, sometimes I would assing the same post two, three or four labels simultaneously.

Then I wrote a whole bunch of keywords and anti-keywords. By "anti-keywords" I mean "if this keyword is in the text/title of the post, then it definitely does NOT belong to this category". Of course, I automated a lot of it. Right now I have over 20,0005 keywords in total.

Then I wrote a simple classifier that checks if the post contains a keyword and outputs a label based on that.

But here's the thing - what if a post has several keywords that belong to several diferent categories? Then it depends on which keyword is checked for first. For example, if a post contains "desired retention" and "10m", if the classifier checks for Desired Retention keywords first, the output will be "Desired Retention", but if the classifier checks for Learning Steps keywords first, then the output will be "Learning Steps".

So in order to maximize the accuracy, I need to check for keywords in a specific order. I had to write my own evolutionary optimizer to do this:

1​)​ Randomly generate a list of numbers that tell the classifier the order in which to check the keywords. Here's what the default ones looks like: `[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21]`. This tells the classifier "First, check the keywords that are related to class 0 (in Python, indexing starts from 0). Then check keywords that are related to class 2, then to class 3, and so on". The goal is to find the order of numbers that maximizes accuracy aka number of posts where the output label is the same as the true label.

2​)​ The output label is compared with the true label to determine the accuracy. Then each list ("specimen" or "members") with numbers ("genotype" where each nubmer is a "gene") will have it's own value of accuracy ("fitness").

3​)​ The bottom specimen (usually around 67% of the first "generation" and around 33% of the last "generation") with the lowest fitness are culled.

4​)​ The remaining specimen will have children. The "parents" are selected in the following way: 3 "mother" candidates are randomly chosen, and the one with the highest fitness is the "mother". Same process is used to select the "father". Thanks to culling the specimen with the lowest fitness and to making specimen with high fitness more likely to have offspring, there is enough optimization pressure (as I like to call it) to cause evolution. Random shuffling doesn't lead to evolution, there has to be some sort of process that makes the less fit less likely to leave offspring and/or makes the more fit more likely to have offspring aka optimization pressure.

5​)​ Their genes are mixed to create the child. The procedure is a bit complicated, so I won't describe it. It involves combining the genes of both parents plus a small chance of a random mutation. This is repeated until the population is back to the same number of members as before the culling, so that each generation has the same number of members.

6​)​ The next "generation" starts. Note that it's possible for a specimen with very high fitness so survive for all generations.

15 generations with a population of 1500 is enough to get good results in my case.

So how accurate is my hand-made classifier? It achieves 65.7% accuracy on the entire dataset. Remember this figure, we'll need it a few thousand words later.

## Part Three: Machine Learning Terminology

(I use "algorithm" and "model" as synonyms)

We want to train a machine learning algorithm on text. Machine learning algorithms don't work with text, they work with numbers. This is a bit of a problem, and by "a bit" I mean **THIS IS A MASSIVE GOD DAMN PROBLEM**. Fortunately, in this case there is a pretty simple way to represent text with a bunch of numbers. Each post will be represented using 27 numbers. The first 21 are the counts of keywords related to each of the 21 categories described above. 5 more for anti-keywords for some categories and "junk" keywords that are unrelated to FSRS in general. And the last number is the standardized length of the post. What is **standardization**, you ask?

(standardization.png)

So after encoding a post might look like this:

`[1.0, 0.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.03288490284005979, 0.3251366120218579]`

All numbers except for the last two are counts of keywords. The last two are the relative position of the first FSRS-related keyword and the [standardized](https://www.geeksforgeeks.org/what-is-standardization-in-machine-learning/) length of the post. I thought that maybe whether the first relevant keyword appears at the beginning of the post or at the end could maybe be helpful.

This is obviously very lossy - the algorithm won't be able to analyze any information about individual words and phrases, all of the semantic nuances are lost. Still, this should be an improvement compared to the hand-made classifier.

Alright, time to feed all this data to the algorithm and enjoy life...is what I would love to say, but we need to talk about **overfitting**. This is one of the most important concepts in machine learning, and it's not exclusive to neural networks. Here's an example:

![image](https://github.com/user-attachments/assets/aa8d0790-56aa-45b9-9ed2-22eeae21a13a)

On the left we are trying to use a straight line to fit clearly non-linear data. Not a good idea. In the middle we are using a 3rd polynomial (or something like that, idk, I'm not the one who made this image), which fits the data quite well. On the right we are using some super high degree polynomial or some really complicated function to fit the data.

- But the fit on the right looks much better! What's the problem?

The problem is that the model on the right won't **generalize** well. Generalization is the model's ability to perform well on previously unseen data aka data that it wasn't trained on. If the model performs great on training data, but outputs garbage when you give it new data, then it's a terrible model. How do you make sure that your model isn't overfitting?

1​)​ Ensure that the number of parameters is many times smaller than the number of datapoints. For example, if your model only has 3 parameters, it's *probably* ok to train it on 20-30 datapoints. However, when it comes to neural networks, it's not uncommon to train gigantic models on relatively small (compared to the number of datapoints) datasets.

2​)​ Early stopping. It's perfectly simple - train the model on the trainining dataset (I will call it "train set") and keep an eye out for its performance on the testing dataset (test set). ![Early stopping](https://github.com/user-attachments/assets/d98cecf1-a56d-4156-be0d-db191805250f)

Once the error on the test loss stops decreasing, stop the training. In practice the curves aren't so smoothed and are more jagged, so we don't stop training immediately and keep it for a few more epochs. An "epoch" is one full pass over the entire dataset. For example, in Anki FSRS is trained with 5 epochs, meaning that it will go over the entire dataset 5 times. More complex models require more epochs to train. This is my preferred method because it's simple and doesn't require a lot of fine-tuning. Typically, around 70-80% of all data is used for training and 20-30% is used for testing. I do 50:25:25. 50% goes into the training set, 25% goes into the test set, and 25% goes into the validation set. I'll explain why I need thatl ast one later. I use 5 folds, meaning that I create 5 different training sets and 5 different test sets. This allows me to more accurately test the model, but it also means that I effectively have to train the model 5 times.

Simplified diagram:

![Transformer training split diagram](https://github.com/user-attachments/assets/e25d709b-e12c-472e-a7f3-d73f6cecd6d2)

"Simplified" because in reality the red and blue bars would look like a glitched TV due to randomization.

3​)​ Dropout. It's basically giving your model some brain damage. During training you randomly set some fraction of parameters to 0. This makes it so that the model cannot learn to rely on specific parameters too much. This requires tuning the percentage of parameters that are randomly set to 0, typically between 10% and 50%, though recent studies suggest that 50% is excessive.

4)​ L2/L1 regularization. Ok, this one is a bit complicated. In neural networks, if a parameter is very large (ignoring the sign, just the magnitude), it's probably a sign of overfitting. We can prevent it by explicitly adding a term for parameter magnitude to the neural network's **loss function** - the stuff that it needs to minimize:

Total error = training error + λ ⋅ |value of the parameter|

|| means "absolute value", λ (lambda) is a hyperparameter that must be fine-tuned. This is known as L1 regularization. L2 regularization looks like this:

Total error = training error + λ ⋅ |value of the parameter| ^ 2

Here the magnitude of the parameter is squared. The main difference is that the former can force the parameters to be exactly 0.

This is my least favorite method because it's a pain to fine-tune.

About the loss function. It represents the difference between predictions and reality, in some mathematical sense. There are lots of loss functions for different problems. All you need to know is that usually "lower = better", and if that's not the case, you can just add a minus sign to your loss function to turn it from "higher = better" into "lower = better".

Finally, let's talk about the **optimizer**. It's the stuff that updates the model's parameters, and calculates the exact amount by which the parameter needs to be updated based on the value of the loss. In most cases the optimizer uses some variant of "gradient descent". Here's the classic illustration:

![image](https://github.com/user-attachments/assets/7ee74e66-8a60-4418-922e-ea0b58d2b935)

("w" is some parameter, "cost" is the loss function)

Imagine a round rock rolling down a ravine. The steeper the slope, the faster it will roll. With gradient descent, the more the loss changes if you change the parameter, the larger the update to the parameter will be. If some parameter has no effect on the loss, it won't be updated. Ideally, we would like to update the parameters once and be done with it, but 99.9% of the time that is not possible and optimization requires several steps.

Here's a cool gif:

![1_hUd744hDEEGx0-ypWGhrkw](https://github.com/user-attachments/assets/997a450a-5715-4c32-bc57-99a3e6c771c3)

I highly recommend you to watch [this video by 1blue1brown](https://youtu.be/IHZwWFHWa-w) for a detailed explanation.

## Part Four: Multi-Layer Perceptron

Initially I wanted to use [MLPClassifier](https://scikit-learn.org/dev/modules/generated/sklearn.neural_network.MLPClassifier.html) from the `sklearn` library, but quickly realized that it's very limited. It was time to ditch the toys and bring out the big guns. It was time to learn Pytorch.

![Pytorch is funny Shawshank redemption](https://github.com/user-attachments/assets/0fc761c9-35d3-40d3-b71c-660bd97a7028)

In case you have no idea what that is - it's library for machine learning. The Python version of FSRS was developed using Pytorch, btw. Not gonna lie, I hated it at first because it's complicated.

Multi-layer perceptron is the simplest neural network architecture. You take your data, for example, just a number x, and you want to get the output number, y. Then you do a linear transformation, x1 = a1 ⋅ x + b1. Then you do a non-linear transformation x2 = f(x1), and there are quite a few ways to do that, more on that later. Then you do a linear transformation again, x3 = a3 ⋅ x2 + b3. Then a non-linear one again, and so on. Basically, you shake and stir your number until it looks like what you want the output to look like. Btw, the dimensionality of intermediate values can be greater than the dimensionality of the input data. In plain English, if your input is just a number x, instead of keeping one intermediate value (like in the example above), you can keep several intermediate values. Like this: x1_1 = a1_1 ⋅ x + b1_1, x1_2 = a1_2 ⋅ x + b1_2, x1_3 =a 1_3 ⋅ x + b1_3, etc. The amount of times you repeat transformations (depth) and the amount of intermediate values you keep (width) are **hyperparameters** - settings of the neural network that determine its structure. You can tweak them manually or use some stuff to optimize looking for the best hyperparameters, more about that later.

## Part Five: Tokenization

After the multi-layer perceptron, I decided to make a Transformer. It's the same architecture that is used in ChatGP**T** ("T" stands for "Transformer"), it's very good for natural language processing (NLP). However, this time I can't use keyword counts, I have to actually convert each word into a number. There are many ways to do that. A simple and commno method is to just assign an integer number (index) to N most popular words and then make the model learn N different **embeddings**. Don't worry, I'll explain what those are later.

For now what matters is assigning integers to words. Here is an example sentence:

`Hey everoyne, I have a problem with FSRS. I can't optimize my parameetrs. akf92j56kcvjk. Screenshot: https:www.https://preview.redd.it/thw6he9n88nc2`

First, let's remove the URL. Long story short, URLs are a pain in the arse. We can remove them automatically:

```python
import re

re.sub(r'http\S+', '', sentence)
```

Result:

`Hey everoyne, I have a problem with FSRS. I can't optimize my parameetrs. akf92j56kcvjk. Screenshot: `

Now we need to split it into **tokens**. A token is not necessarily a word, it could be a period, a comma, a semicolon, a number, etc. Thankfully, we can use the greatest Python technique known as *Import That One Library That Does Exactly What I Want*, or "ITOLTDEWIW".

![Import That One Library That Does Exactly What I Want rainbow](https://github.com/user-attachments/assets/fd471b6f-9f3c-47c0-aab5-1e7bf06b9548)

Ok, there is probably a better name for this...Anyway.

`from nltk.tokenize import word_tokenize`

Now our sentence has turned into...

`['Hey', 'everoyne', ',', 'I', 'have', 'a', 'problem', 'with', 'FSRS', '.', 'I', 'ca', "n't", 'optimize', 'my', 'parameetrs', '.', 'akf92j56kcvjk', '.', 'Screenshot', ':']`

...a list of strings!

("string" just means "a bunch of text characters" in proggrammerspeak)

Notice that "can't" becomes two different tokens and spaces are removed. I'll have to add an extra rule to turn "ca" into "can".

Now let's make everything lowercase. Why? Because it avoids situations such as "fsrs" not being equal to "FSRS" and overall simplifies a lot of things. Here's the result:

`['hey', 'everoyne', ',', 'i', 'have', 'a', 'problem', 'with', 'fsrs', '.', 'i', 'can', "n't", 'optimize', 'my', 'parameetrs', '.', 'akf92j56kcvjk', '.', 'screenshot', ':']`

Nice. However, what is that gibberish near the end? We don't want our neural net to work with junk. So I wrote a pretty sophisticated function to check if a string is random gibberish or not. After applying it:

`['hey', 'everoyne', ',', 'i', 'have', 'a', 'problem', 'with', 'fsrs', '.', 'i', 'can', "n't", 'optimize', 'my', 'parameetrs', '.', '.', 'screenshot', ':']`

Better! But some words are misspelled. It's time to use the greatest Python technique again.

```python
from spellchecker import SpellChecker

spell = SpellChecker()

word = spell.correction(word)
```

Result:

`['hey', 'everyone', ',', 'i', 'have', 'a', 'problem', 'with', 'furs', '.', 'i', 'can', 'not', 'optimize', 'my', 'parameters', '.', '.', None, ':']`

We've got a few problems. It corrected "fsrs" to "furs" and "screenshot" to...None. Not a string "None", but None (that's a Python thingy). So we need to add a few exceptions.

`['hey', 'everyone', ',', 'i', 'have', 'a', 'problem', 'with', 'fsrs', '.', 'i', 'can', 'not', 'optimize', 'my', 'parameters', '.', '.', 'screenshot', ':']`

Ok, now it's time for the final step before converting tokens to numbers: **lemmatization**. Lemma is a...uhhhh...let me just give you some examples.

1​)​ walk, walking, walked -> walk

2​)​ was, is, be -> is

3​)​ mice, mouse -> mouse

Time to use the greatest Python technique again. Code:

```python
from nltk.stem import WordNetLemmatizer

from nltk import pos_tag

from nltk.corpus import wordnet
```

```python
def f_lemmatize(token: str):
    def get_wordnet_pos(treebank_tag):
        if treebank_tag.startswith('J'):
            return wordnet.ADJ
        elif treebank_tag.startswith('V'):
            return wordnet.VERB
        elif treebank_tag.startswith('R'):
            return wordnet.ADV
        else:
            # Default to noun if no match is found or starts with 'N'
            return wordnet.NOUN

    lemmatizer = WordNetLemmatizer()
    thingy = pos_tag([token])
    if len(thingy) == 0:
        return token
    elif thingy[0][1] == 'NN':
        return token
    else:
        tag = thingy[0][1]
        lemma = lemmatizer.lemmatize(token, get_wordnet_pos(tag))
        lemma_no_tag = lemmatizer.lemmatize(token)
        if lemma != token and lemma is not None:  # if lemma is different from the token and isn't None
            return lemma
        elif lemma_no_tag != token and lemma_no_tag is not None:  # try it again, but without the tagging function
            return lemma_no_tag
        else:  # just give up
            return token
```

Final result:

`['hey', 'everyone', ',', 'i', 'have', 'a', 'problem', 'with', 'fsrs', '.', 'i', 'can', 'not', 'optimize', 'my', 'parameter', '.', '.', 'screenshot', ':']`

Notice that "parameters" turned into "parameter". Lemmatization can help a lot, especially if I add extra rules manually. This is lossy - we lose some of the semantic nuances. However, I don't have a ton of data. If I had 100,000 posts - sure, I could afford to treat each word individually. But with only around 1k posts the neural net won't have enough data to learn all the nuances, so it's better to use as few tokens as we can.

After adding ten gorillion extra rules and exceptions I finally managed to make everything work (mostly) the way I want. Of course, as I gather more data, I will add more rules and exceptions.

Now all that's left is to assign an integer to every word. It doesn't really matter how you do it, but I liek doing things in a way that makes sense, so I did the following:

1​)​ 0 is a special integer reserved for padding ("pad" as I call it). You see, we need to make every text the same length (in tokens). I chose 512 as maximum length. A text is longer than that? Truncate it, then assign integers. A text is shorter than that? Assign integers then add a bunch of zeros as the end.

2​)​ Every token is assigned an integer based on its frequency in the dataset. Most common token will be 1, second most common will be 2, third most common will be 3, etc.

3​)​ If a token appears <4 times in the entire dataset + "GPT'ed" dataset (more about that later), it will be assigned a special integer reserved for obscure crap and typos ("unk"). A token must appear at least 4 times across three variations of the dataset to warrant having it's own index.

The overall vocabulary size of my Transformer is currently 6448 unique tokens. For ~~magical~~ programming reasons, I made it a multiple of 8 (as well as a few of other things, like text length and some hyperparameters). Minus 0 because it's for padding, minus 6511 because it's for obscure crap and typos. 

## Part Six: Can I Get More Data?

NLP models require a lot of data, but at the time I only had around 600-700 posts scraped.

Can I get more data?

Well, I can make the scraper look for older posts and downvoted posts at the cost of making it slower. That increased the number to around 800-1000 (as I said, I wasn't keeping track of everything precisely).

Can I get more data?

I can make the search even more exhaustive and slower to scrape more posts. Also, I can add posts that have absolutely nothing to do with FSRS whatsoever, to help the model learn to better differentiate between relevant and irrelevant posts. After some tweaking, I managed to get around 1400-1450 posts.

Can I get more data?

I guess it's time to scrape comments now. However, most comments aren't useful since there are too many comments where the person is explaining FSRS rather than asking a question about FSRS. I need questions, not answers. So I only labeled 92 comments out of several thousands.

*Can I get more data?*

I can't get any more data...or can I? It's time to learn about another important concept: **data augmentation**. By taking the original data and tweaking it, we can create more training examples and make the neural net robust to relatively small differences that wouldn't throw a human off. Here are some examples of what is used in computer vision tasks:

![Data Augmentation kitten v1 1](https://github.com/user-attachments/assets/da72d984-04c2-4c54-9e23-79ea840d9674)

Of course, the techniques used for text are different from those used for images. The first data augmentation technique that is used is rephrasing using LLMs, I fed all 1,519 texts to GPT-4o-mini and asked it to rephrase them for me. Example:

![GPT-4o-mini rephrasing](https://github.com/user-attachments/assets/e78b49ba-1e5a-4be5-84f5-6b6f91f03b3f)

I will be using 50% of the dataset for training, so only 0.5*1519=1,063 texts will be used for training.

Rephrasing trippled the size of the training set, from 1,063 texts to 3,189 texts.

**Can I get more data?**

I'm not sure if this technique has a name, so let's call it "fusing". Key idea: if two posts have the same label and their combined length, in tokens, is <= the context window of the Transformer (which is 512 tokens in my case), we can smash them together to create a third one. If you combine the text labeled as, say, "Helper Add-on", with another text that is also labeled as "Helper Add-on", then obviously the combined text should have the same label. Why wouldn't it?

From a human perspective, this can result in posts where it looks like the first part was written by one person with one issue, and the second part was written by a different person with a different issue. Still, I expect it to be useful as an augmentation technique. This is also the slowest, most time-consuming one.

This doesn't work on posts that are too long to fit into the Transformer's context window after being fused with other posts, and also on posts that have 2-3 labels and this combination of labels is unique and doesn't occur anywhere else in the dataset. Because of all that, the overall increase in the number of training examples is less than 2.

Fusing texts increased the size of the training set by roughly x1.92, from  texts to  texts. Note that unlike in the example with the kitten that I showed above, I apply multiple augmentation techniques sequentially to get even more data, so that the size of the dataset increases exponentially as I add more techniques.

**Can I get more data?!**

I can take existing texts and randomly swap two adjacent (one comes after the other) sentences. Example:

'I went from having 100 reviews to having 300 reviews every day. I am seeing the same cards over and over again.' -> 'I am seeing the same cards over and over again. I went from having 100 reviews to having 300 reviews every day.'

Initially, I tried writing some really complicated regex stuff, but then I realized that I can just do this:

```python
list_of_sentences = nltk.sent_tokenize(text)

list_of_sentences = [(x + ' ') if (x != list_of_sentences[-1]) else x for x in list_of_sentences]  # add whitespaces to everything except for the last sentence
```

I made it so that if a text has 2-5 sentences, two randomly chosen adjacent sentences would be swapped. If the text has >5 sentences, four sentences (two pairs) will be swapped.

Sentence swapping doubled the size of the dataset, from 4,400 texts to 8,800 texts. Sure, short texts with just one sentence are duplicated, by meh, whatever.

***Can I get more data?!***

This next technique is my own invention, I haven't seen it in literature. I call it "filler sentence injection". First, I write down a bunch of filler sentences, such as "Hello everyone", "Hi", "EDIT: added screenshots", "P.S. English is not my native language", "Help would be appreciated", "What are your thoughts, fellow Anki users?", "I would like to hear from experts", "I'm not 100% sure", etc. These sentences don't change what the text is about. If a text is about learning steps, it will be about learning steps with or without these sentences. If a text is about Easy Days, it will be about Easy Days with or without these sentences, etc. Then I randomly inject one of these sentences inbetween two other sentences, or before the first sentence, or after the last sentence. For the sake of keeping it similar to a text actually written by a human, some filler sentences like "P.S. I love this community!" are only appended at the end, and some, like "Greetings, everyone!" are inserted only in the beginning. Obviously, nobody *starts* their post with P.S.

I did three rounds of filler injection to obtain three more variations of the dataset and it quadrupled the size of the dataset, from 8,800 texts to 35,200 texts.

<ins>***CAN I GET MORE DATA?!***</ins>

Ok, it's time for the final technique. What if instead of modifying the text, we modified the indices? Here's how exactly:

1​)​ Index of a valid token -> index of "unk". Imagine that someone misspelled a word and my spellchecker didn't catch that. In that case the word would turn into something that isn't a valid word, hence it would be assigned the "unk" index. So if make it so that there is a small probability of an index randomly turning into the "unk" index, we can simulate uncorrected typos.

2​)​ Index of a valid token -> index of a valid token. For example, someone may have typed "internal" instead of "interval", or "stage" instead of "state". Both are valid words, so the spellchecker won't do anything. How do we decide what will be the replacement? Time for another cool python library: [https://github.com/MaxHalford/clavier](https://github.com/MaxHalford/clavier). It allows you to measure the distance between words - in the "distance your fingers have to travel" sense - for a given keyboard layout. I chose this layout:

![image](https://github.com/user-attachments/assets/0b342669-6a24-49e3-86bd-8b2027a95242)

Then for each word in the dataset I measured its distance to each other word to find its nearest neighbor, like "interval" -> "internal". This way we can simulate a different kind of typo, the kind that a spellchecker can't possibly catch.

Then I assigned a 4.8% probability to 'index of a valid token -> index of "unk"' and a 1.7% probability to 'index of a valid token -> index of a valid token'.
That's a total 6.5% probability of a typo *per token*, or approximately 99.88% probability of at least one typo per 100 tokens.

Then all I had to do was just run the randomizer 4 times to create 4 more variations of the train set. This brought the total number of texts in the training set to **140,800**, 133 times more than without data augmentation!

The total number of tokens for training is around 36 million. For comparison, that's around 14 thousand times less than what GPT-3 was trained on, and around 374 thousand times less than what GPT-4 was trained on. Granted, they do different things - classifying entire texts vs predicitng the next token - but I just wanted to give you a sense of scale.

So to summarize: I rephrased the texts using GPT-4o-mini, I combined some texts to create new ones, I swapped some adjacent sentences, I added filler sentences, and finally I simulated typos that turn valid tokens into crap as well as typos that turn valid tokens into other valid tokens. This is all for the train set, the test set consists of non-augmented, original texts.

![image](https://github.com/user-attachments/assets/034adb93-ff26-40e3-9f6f-16389e692bdf)

Comparison of different data augmentation methods that I used:

![image](https://github.com/user-attachments/assets/56baa708-9c1b-4552-8738-7295d48bc6b9)

By "diversity" I mean "variation between the original and augmented data".


Oh, and I also added some completely made-up texts for very rare classes, such as "Automatic Optimization". If you don't have enough real data, add half-fake data. If even that is not enough, add fake data. As long as you can manually make high-quality examples, it's fine.


**IMPORTANT**: when doing data augmentation, make sure that the test set doesn't have any variations of texts that are in the train set, or else the model will display unrealistically good results on the test set only to shit itself in real life. It's called data leaking. In other words, if there are N variations of text X, make sure that **all** N variations stay in the train set and **none** of them are in the test set.

___
### [←Return to homepage](https://expertium.github.io/)
