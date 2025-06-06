# Traveling Salesman Problem: an example where "Nearest Neighbor" results in the WORST possible route

Traveling Salesman Problem (TSP  for short) is one of those problems that Randall (xkcd author) would definitely classify as "weirdly concrete".

![image](https://github.com/user-attachments/assets/2d1f06e6-1dc6-4330-9595-add1d36c488d)

The problem is simple: you have N cities, each with some coordinates, and you need to find the shortest route such that you visit every city only once and then return to the starting city.

Example: 

![image](https://github.com/user-attachments/assets/ab22708f-f68c-48ec-917f-2003a4f209c0)

Sounds easy, right? Well, as it turns out, no. You could try every possible combination of routes, but doing that becomes too computationally expensive very quickly. And the algorithms that do not try every possible route are complicated.

*"Just always go to the nearest city, duh"*

This sounds nice, but somewhat unintuitively, in practice it often results in a suboptimal solution. What's far more unintuitive is that Nearest Neighbor (NN for short) - "just go to the city that is closest to the current city" - can actually result in **provably THE WORST possible route**. Not just suboptimal. THE WORST.

When I first heard that, I was really intrigued. How can we construct an example like that? I did a whole lot of googling and the only useful piece of information that I found was this comment on math.stackexchange:

![image](https://github.com/user-attachments/assets/2183bf5c-a5b0-4b29-9939-ad5ecedd7ebd)

I thought, "Alright, I've got the distances, so surely finding the coordinates will be easy, right?"

...right?

Well, after spending 2 hours doing a bunch of stuff with circle intersections in Photoshop, I gave up and asked Claude 4 Sonnet to implement some code to do it for me. As it turns out, it's actually impossible to construct an example with these specific distances.

Here are the coordinates that I ended up with:

```
cities = [(0, 0), (200, 0), (101.002500, -173.780019), (398.997500, -28.301855)]
```

And if you do the math:

  1. Route: 1 -> 2 -> 4 -> 3 -> 1, Distance: 933.6096
  2. Route: 1 -> 3 -> 4 -> 2 -> 1, Distance: 933.6096
  3. Route: 1 -> 3 -> 2 -> 4 -> 1, Distance: 1002.0000
  4. Route: 1 -> 4 -> 2 -> 3 -> 1, Distance: 1002.0000
  5. Route: 1 -> 2 -> 3 -> 4 -> 1, Distance: 1131.6096
  6. Route: 1 -> 4 -> 3 -> 2 -> 1, Distance: 1131.6096

1 -> 2 -> 3 -> 4 -> 1 is the route that NN would choose.

Finally, here's a visualization:

![Figure_1](https://github.com/user-attachments/assets/09af3e79-d8ab-4658-9ed0-5243ff068e8c)

Suppose you start in city 1 (though the starting city doesn't affect the result).

1) The closest city to city 1 is city 2 with coordinates (200, 0) and the distance of 200 units from city 1. So you would go to city 2.
2) The next closest city is city 3 with coordinates (101.002500, -173.780019) and the distance of approximately 200 units from city 2. So you would go to city 3.
3) The next closest city is city 4 with coordinates (398.997500, -28.301855) and the distance of approximately 331.6096 units from city 3. So you would go to city 4.
4) Then, finally, you would go back to city 1. The distance between city 4 and city 1 is approximately 400 units.
5) Then the total distance that you have traveled would be approximately 200 + 200 + 331.6096 + 400 = 1131.6096. And if you look at what I wrote above, that is the longest possible route!

I love this example because of how counterintuitive it is. NN not only results in a suboptimal route, but it results in the worst possible route. I thought this would require a lot of cities arranged in some weird geometric pattern and was surprised that such an example is possible with just 4 cities.

Special thanks to Claude 4 Sonnet and user856 on math.stackexchange.

___
### [←Return to homepage](https://expertium.github.io/)
