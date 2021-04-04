# Shortest Paths

I was wondering for a while simple ways on how to generate roots or veins structures that possibly didn’t rely on solvers. There is a very interesting node in Houdini called FindShortestPath which will basically search for the most optimal path between one or multiple start and end points on a given mesh.

This is a rather simple setup, but I wanted to share it nonetheless. If you’re interested in these kind of growth effects, check out the amazing work of [Richard Lord](https://www.richlord.com/), which was quite an inspiration for these explorations of my own.

The challenge here was to have interesting meshes to let the node onto run in the first place. Depending on your topology you can achieve different results. By having a geometry composed of quads, it’s easy to generate interconnected grids, or even”pipe-like” structures. But my goal was to achieve a more organic look.

![roots_shortest-path](https://user-images.githubusercontent.com/81909946/113515466-fe4aec00-9574-11eb-8348-2a077e6e64f6.gif)

A solution I found for this was by fracturing my geometry with the voronoi algorithm before looking for the shortest path between any points. By scattering points onto a varying density volume, for example, some very interesting source points configurations start to emerge, and the results are quite convincing.

A cheap way to fake obstacles is to simple boolean out pieces of the geometry where nothing should grow after fracturing it. Check out the example file below if you are interested in seeing it for yourself. There are a couple nice tricks I like in there, such as a radial noise generation for controlling the scattered points (both as a point and volume VOPs).

Anyway, hope you enjoyed it! Also, here’s my new desktop background (click for 4k resolution).

![particula_shortest-path_wallpaper](https://user-images.githubusercontent.com/81909946/113515481-0e62cb80-9575-11eb-8ece-f79ee1a20111.jpg)


