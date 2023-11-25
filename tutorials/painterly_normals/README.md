# Painterly Normals

![painterly-normals_02](https://github.com/ribponce/particula/assets/81909946/131936db-012c-4e16-9f6d-8dfca8cb6e10)

Let's have a look in Houdini at how to make a procedural painterly normals setup, recently made famous by [Cody Dingy](https://www.youtube.com/watch?v=s8N00rjil_4).
This is a very basic implementation, but I believe it's still quite interesting to be shared.

![suzanne_N](https://github.com/ribponce/particula/assets/81909946/c62216e2-8d94-41f1-915f-124f6bcf062c)

You may notice the resulting Normal texture kind of looks like a voronoi pattern. This is due to the usage of the XYZDIST function inside the VOP generator in COPs.
It's basically sampling the closest point, and since the distance is set to a large number, it generated this flood fill effect.

To make the normal texture in Houdini look like a normal texture coming out from blender we need to do a few operations.
Due to the different coordinate system, we need to swizzle Y and Z, as well as invert Z in the process. They all have to be remapped to a 0-1 range as well.

![image](https://github.com/ribponce/particula/assets/81909946/5520f806-0d5d-40a8-80b5-f4cc1ad0c5d6)

Inside the COP network you will notice a couple switches. This is more of an stylistic choice, bet we may also try to blend the painterly normals generated from the procedural "strokes" with the original normal map. I'm using the Maximum composite operation, and that's decent but I wish there was a better way of blending them. If you can think of something better please let me know.
By not multiplying the generated texture with the original UV mask we also get rid of some black seams in the shaded model, so that's also something to be aware.

![painterly-normals_03](https://github.com/ribponce/particula/assets/81909946/283176ac-65ef-41ba-98e8-17294dba8294)

Lastly, here's the overview of the generation. As I mentioned it's a very simple approach, and has a lot of room for improvement. Currently only simple grids are used, so it would be possible to perhaps generate some brush strokes and sample those somehow.
Some changes to the sampling method might be required though.

![painterly-normals_04](https://github.com/ribponce/particula/assets/81909946/062f78cc-3ec0-4b87-aa14-61ef2a1309db)

It's also worth noting, as seen in the video, a similar (and arguably better) procedural approach can be achieved with Substance Designer. Nonetheless I think this was a neat thing to expore in Houdini.

That's all I have for you today. Thanks!

[Download the .hip file here](https://github.com/ribponce/particula/blob/master/tutorials/painterly_normals/files/particula_painterly-normals_SHARE.hip)

[Download the .blend file here](https://github.com/ribponce/particula/blob/master/tutorials/painterly_normals/files/particula_painterly-normals_SHARE.blend)
