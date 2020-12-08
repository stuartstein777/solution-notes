# Advent of Code 2020 Day 7

The puzzle link:
https://adventofcode.com/2020/day/7#part1

## Part 1
This puzzle involves generating calculating the number of ways we can arrive at a shiny gold bag when other bags can contain shiny gold bags.

Example test input:

```
light red bags contain 1 bright white bag, 2 muted yellow bags.
dark orange bags contain 3 bright white bags, 4 muted yellow bags.
bright white bags contain 1 shiny gold bag.
muted yellow bags contain 2 shiny gold bags, 9 faded blue bags.
shiny gold bags contain 1 dark olive bag, 2 vibrant plum bags.
dark olive bags contain 3 faded blue bags, 4 dotted black bags.
vibrant plum bags contain 5 faded blue bags, 6 dotted black bags.
faded blue bags contain no other bags.
dotted black bags contain no other bags.
```

If we want to carry a shiny gold bag the options are:

1) Put it in a bright white bag
2) Put it in a muted yellow bag
3) Put it in either a bright white bag or a muted yellow bag, inside a dark orange bag.
4) Put it in a bright white bag or a muted yellow bag, inside a light red bag.

So we have four options.

Given the full puzzle input, how many bag colors can we choose to carry our shiny gold bag?

This can solved using a directed weighted graph. The example test input as a directed weighted graph would look like:

![Final solution in REPL](aoc2020day7/fig3.png)

From the above you can see the muted yellow node connects to the shiny gold. As does the bright white node.

If I want to find how many ways I can arrive at shiny gold, from any node, I need to transpose the graph. Which for a directed weighted graph means reversing each directed edge.

The transposed test graph:


