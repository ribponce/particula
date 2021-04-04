# Procedural Modules

[Watch on Youtube](https://www.youtube.com/watch?v=MwFARaaTLW4)

![Procedural Modules](https://user-images.githubusercontent.com/81909946/113516073-a0b89e80-9578-11eb-9853-840f3b6a8ba5.jpg)

Being able to copy any number of primitives from a selected pool to *n* points efficiently is something that comes up really often. Of course there are many ways of tackling any given problem, but this approach gives us the ability to further implement other core concepts that end up being essential for developing realistic modules.

Humans are extremely good at perceiving repeating patterns. A simple copy-paste is rarely enough to trick our brains into thinking something looks real. With that in mind, I researched many different ways of creating unique modules when modelling procedural buildings. This quick video shows one of the core principles that I developed that allowed in a relatively cheap way to generate an infinite amount of combinations when instantiating walls, windows and balcony modules.

The wrangle for creating our index attribute is simple:

```c#
i@index = (int)fit01(rand(@ptnum, ch("seed")), 0, ch("max"));
```

At the Switch operator we simply use a point function. Being able to access each of the incoming points separately inside the loop is what makes this setup work in the first place. I think it’s also worthwhile to mention the clever solution to make the whole thing even more procedural; not having to manually change the amount of primitives connected to the Switch node is quite handy and solved with the opninputs() function.

Another nice addition to this (which hasn’t been shown nor implemented) would be to add weighting. In some cases it could be interesting to have the ability to make some primitives have a higher or lower probability of being instantiated.

A simplified scene file shown in the first part of the video is available for [download here.](https://github.com/ribponce/particula/blob/851e570a8305e3a82b6d94f043f6dd402b869749/tutorials/procedural_modules/files/particula_procedural-modules_SHARE.hipnc)

I haven’t included the shape generation part of the setup, since I think it deserves to be first fleshed out to a more polished state. I might revisit it again soon.

Anyway, that’s it for now. Thank you, and I’ll see you in the next one!
