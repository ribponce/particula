# Falloff from points

Creating falloffs is useful in countless situations. The base for this snippet came from a presentation by Entagma, where they showcased a bunch of cool projects. Rebuilding setups and playing around with other people’s solutions are one of the best ways to learn. We run the code over points.

```c#
int pntcnt = npoints(1);
float accum = 0;
for(int i=0; i<pntcnt; i++)
{
    vector tarpos = point(1, "P", i);
    float dist = distance(tarpos, @P);
    float falloff = fit(dist, ch('start') , ch('end') , 1,0);
    accum = max(falloff, accum) ;
}
@Cd = set(accum, 0, accum/4);
```

![falloff-from-points](https://user-images.githubusercontent.com/81909946/113508684-a8188180-9551-11eb-84bf-bc4a3d51817e.gif)
![falloff-from-particles](https://user-images.githubusercontent.com/81909946/113508638-71426b80-9551-11eb-8505-bc7f48d15560.gif)

Nice thing is that particles are also points, so we can implement some sort of decay when using them as source. We simply divide a particle life attribute by its age and multiply it by the falloff (both of those attributes are created when simulating particles on a pop/dop network).

```c#
float decay = point(1, "life", i)/point(1, "age", i) * 0.1;
accum = max(falloff*decay, accum);
```

The same principle is applied to ‘activate’ particles from a grain simulation. Just as an example that multiple simple setups always end up coming together to achieve bigger and better results. Check out the scene file below for further details, but in a nutshell, after caching the first frame of the grain simulation (given that we only create points at the first frame), we create a separate particle system which spawns a few points. We then use those points to create a falloff when getting closer to the first-frame-cached points from the grain simulation. The attribute is then transferred back into the grain sim and sets the @stuck attribute to zero, basically releasing them to be influenced by forces inside the dop. In this case the color red is only mirroring the @active attribute we created, for visualisation purposes.

![grains-activ](https://user-images.githubusercontent.com/81909946/113508719-d1391200-9551-11eb-9340-53fc48764837.gif)

[Download simple falloff example scene file.](https://github.com/ribponce/particula/blob/71d3cc211ec9899675288598524ac31a1e9007be/files/vex/particula_falloff-from-points_SHARE.hipnc)

[Download grain particle activation scene file.](https://github.com/ribponce/particula/blob/71d3cc211ec9899675288598524ac31a1e9007be/files/vex/particula_grain-particle-activation_SHARE.hiplc)



# Group/delete primitives by percentage

Append a Sort SOP before this to randomize the primitive numbers. Based on a slider, a certain percentage of your incoming primitives will get either deleted or grouped. By dividing the current primitive number by the total number of prims we basically order them in a range from 0 to 1. Quite simple but really handy. Run over primitives.

```c#
float del = (float)@primnum/@numprim;
if(del<ch("amount"))
{
removeprim(0, @primnum, 1); //use this line to delete the prims
//@group_del = 1; //or this to only group them
}
```

A more direct approach would be to invert the order of the operations, and declare the threshold variable from within the if statement.

```c#
if(@primnum<=(@numprim*ch("amount"))){
@group_del = 1;
}
```


# Densely packing circles

From a post at the odforce forums, asking how to optimally pack circles from points scattered onto a surface. First approach was to solve it in VOPs, but the VEX solution later seemed to be much more elegant. In principle, we want to evaluate each point and its closest neighbour, store half of the distance between them and define that this should be the biggest radius possible for the circle copied to it. Simple way to do it is to write it to the @pscale attribute. Run it over points.

```c#
int pcloud = pcopen(0, "P", @P, chi("radius"), 2);
vector found = pcimportbyidxv(pcloud, "P", 1);
@pscale = length(found - @P)*.5;
```

![pack-circles](https://user-images.githubusercontent.com/81909946/113509196-879df680-9554-11eb-90d5-f60beb1cb6f0.gif)

Point clouds are really interesting. We open a cloud handle for each point, set a large enough search radius, and the max amount of points to 2 (this function always counts itself, and we only want the first found point). Import by index V, since we want the position vector of that point. That *.5 is our threshold, we can make it larger if we deliberately want the circles to intersect, for example. We can also fill up the spaces more evenly tweaking the relax parameters on the Scatter. As long as we have proper normals, this will work with any geometry.

![pack-circles-pig](https://user-images.githubusercontent.com/81909946/113509198-8967ba00-9554-11eb-8108-f797e8c01dfe.gif)


# Connecting dots

Say we want to add lines between points based on their distances. There are of course built-in nodes that could achieve that, such as ConnectAdjacentPieces, but depending on the situation a custom solution might be faster or even your only option. Nearpoints() stores in an integer array a list of the closest points, with the option to specify the maximum amount of points to be found. We evaluate inside a for-each loop every element of the array separately. The if condition is to avoid overlapping primitives. Great thing is that creating geometry has become easier since version 16 (maybe 16.5?). Houdini has gotten smarter and no longer requires us to explicitly add vertices beforehand. Run it over points.

```c#
int npts[] = nearpoints(0, @P, 1.5, 6);
foreach(int pt; npts)
{
    if(pt >= @ptnum && pt != @ptnum){
        addprim(0, "polyline", @ptnum, pt);
    }
}
```


# For Loop and Timeshift

The previous example allows us to freely drape any curve, which we can expand in many different ways. For example, what if we want to copy these curves repetitively and offset their drape amount slightly, plus animate the  whole thing? I learned a similar trick from cgwiki, utilising Timeshift in combination with a For Loop. Here is the overall workflow for such setup:

1. Have an animated piece of geometry that we want to copy and offset. In this particular case it’s a simple sin() function at the amount parameter we created at the wrangle above, taking $F (the current animation frame) as argument.
2. Append a for loop, set it to fetch input, method by count and merge each iteration. Create meta import node. We can now refer to this node to access our loop information.
3. We timeshift the input using the detail attribute from the meta node as argument, added to the current $F. We also transform it in some arbitrary direction right after using the same principle.

In this case we are simply using the iteration number as ‘step’ for both operations, just for the sake of simplicity. We could definitely fit the value to a new desired range, or manipulate it however we need it.

![draping-cables](https://user-images.githubusercontent.com/81909946/113509724-71de0080-9557-11eb-80c9-9553d4af4917.gif)

[Download example scene file.](https://github.com/ribponce/particula/blob/71d3cc211ec9899675288598524ac31a1e9007be/files/vex/particula_draping-cables_SHARE.hipnc)