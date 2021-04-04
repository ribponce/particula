# Interactive Python States

[See on Youtube](https://youtu.be/DTReVsTmKNY)

In this tutorial we take a look at python viewer states inside Houdini 18, and how its practicality can help technical artists develop interactive, easy-to-use tools for fast and smart content creation.

The first iteration of this project was an attempt at replicating by Simon Verstraete’s great results on his article at 80lv. After a few messages he was able to point me in the right direction. So I really appreciate that, thanks Simon!

However, after getting a hang of how things could be implemented, I soon realized that simply extruding geometry wasn’t going to achieve the results I envisioned.!

![cubes_py_demo01_reduced](https://user-images.githubusercontent.com/81909946/113513914-208c3c00-956c-11eb-835d-8964c1419ac6.gif)
*This is where all started. On the first iteration of this project the only accepted input was a flat grid. It also relied solely on extruding primitives, which later forced me to search for other alternatives.*

The reason for that is also mentioned briefly in the video, but to be more specific, I wanted the little boxes to always obey a master grid, so whenever I performed a click for the creation of an overhang (boxes that are connected only by its horizontal neighbor), its shape would also perfectly match with the column of cells below it.

The way I handled that was to start working with a pre-constructed grid, and any actions performed would simply check for the corresponding cell on the grid and merge it back to the original geometry.
