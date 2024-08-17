# Button usage and review time of Anki users

Apart from being great for benchmarking algorithms on it, the [Anki 20k dataset](https://huggingface.co/datasets/open-spaced-repetition/FSRS-Anki-20k) also provides a definitive answer to an age-old question: is the amount of time spent reviewing a card correlated with which answer button (Again/Hard/Good/Easy) the user will press?

Here's the answer: 

![Average reveiw time](https://github.com/user-attachments/assets/502974f1-24cc-4c68-81e6-47965fffefec)

This is based on 20k collections and 1.4 **billion** reviews. The black caps are 99% confidence intervals, but they are too narrow to see properly.

I also analyzed how often the following inequality (and others) holds true: average_t(Again) > average_t(Hard) > average_t(Good) > average_t(Easy).

Here's a stacked bar chart aka [pie chart's cooler brother](https://github.com/cxli233/FriendsDontLetFriends?tab=readme-ov-file#10-friends-dont-let-friends-make-pie-chart):

![Stacked bar graph, 1 1](https://github.com/user-attachments/assets/952020cb-0183-49b0-9329-e9f121e152fd)

In case you are confused: for example, Again > Hard > Good > Easy means "Average time for 'Again' is greater than the average time for 'Hard', which in turn is greater than the average time for 'Good', which in turn is greater than the average time for 'Easy'". But that's too long, so I just wrote it as Again > Hard > Good > Easy.

I hope nobody will interpret this article as "It's ok to use review time to automatically select the answer button for the user".


___
### [←Return to homepage](https://expertium.github.io/)
