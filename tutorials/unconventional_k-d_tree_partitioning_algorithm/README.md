# Unconventional k-d tree partitioning algorithm

## Part 1

[Watch on Youtube](https://www.youtube.com/watch?v=IZoUdVS2Rw0)

Traditional K-dimensional algorithms will normally use neighbouring points to define the search trees and set its constraints. In this unconventional approach, we instead randomise the position of every perpendicular hyperplane that will split the current primitive into two at every iteration of a loop, for as many times as we want.

This highlights interesting applications to the Carve SOP. There are a few other interesting tips and techniques to be learned from here that can also be easily expandable, and I hope this may somehow be of help or even entertainment to you.

## Part 2

[Watch on Youtube](https://www.youtube.com/watch?v=3IfxJ9COIbw)

In the second part of this mini tutorial series I demonstrate a few approaches to handle random extrusions for our space partitioning algorithm.

These techniques are easily expandable and can be adapted to all sorts of projects. The first technique covered utilities Sort SOP’s built-in functionality to randomize the primitive order, while we subsequently perform extrusions inside for-each loops, assigning random values based on the current iteration detail attribute.

Next we go through a simple wrangle to handle the randomization of the elements, utilizing a little bit of vex. This allows us to perform the whole operation inside a single for-each loop.

Last but not least we utilize a Measure SOP to create a system that sorts the primitives based on their areas, that could personally render the nicest results.
