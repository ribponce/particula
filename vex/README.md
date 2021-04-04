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

[Download simple falloff example scene file.](https://github.com/ribponce/particula/blob/b4aa077ec8c758d0b0d386931ac16a4f25ca3a1f/vex/files/particula_falloff-from-points_SHARE.hipnc)

[Download grain particle activation scene file.](https://github.com/ribponce/particula/blob/b4aa077ec8c758d0b0d386931ac16a4f25ca3a1f/vex/files/particula_grain-particle-activation_SHARE.hiplc)




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

[Download example scene file.](https://github.com/ribponce/particula/blob/b4aa077ec8c758d0b0d386931ac16a4f25ca3a1f/vex/files/particula_draping-cables_SHARE.hipnc)




# Growing Spirals

It’s good practice to declare your variables beforehand whenever you can. However, some variables being used inside a loop should be declared from within the loop, in case we might want to perform operations with i at every iteration. It depends on what we are trying to achieve. This is a nice example because it has both.

It’s easy to understand if you break it down. A clever technique is to seek for the functions that are actually ‘doing’ something, and investigate backwards where their arguments are being referenced from and what are they exactly computing. For example, in this case we are basically creating points and connecting them with a line, each with the last one; you can spot both addpoint() and addprim() really easily. The second argument for adding a point is also a function, where we set() a position vector for each iteration of our loop. You see that our angle variable is being multiplied by i, so there is definitely going to be some sort of incremental procedure happening.

```c#
int count = chi("count");
int div = chi("div");

for(int i=0; i <= count; i++){
    float radius = i * ch("radmult");
    float d = 360.0 / div;
    float angle = radians(d*i);
    
    int pt = addpoint(0, set(radius * cos(angle), 0, radius * sin(angle)));
    addprim(0, "polyline", pt, pt-1);
}
```

The snippet above runs in detail mode, and only creates a spiraling line with some controls after promoting the channel parameters. There is a second wrangle that adds some displacement to the lines based on their position; it’s a simple anoise() function, which creates a our beloved alligator pattern. If I’m being honest, in most of the cases where I want to use any sort of noise I’ll choose VOPs over vex wrangles for the convenience and overall better control,  but these functions definitely have their value in this context as well.

![grow-spiral](https://user-images.githubusercontent.com/81909946/113510441-23cafc00-955b-11eb-9a97-7db0f251d833.gif)

[Download example scene file.](https://github.com/ribponce/particula/blob/25c22918431b8b8b9e271b59f379d85a1f597a35/vex/files/particula_grow-spiral_SHARE.hipnc)

# Growth Rings

A similar setup as the Growing Spirals example, where we expand upon the vex code and modify some core principles. First thing you observe is that we have a loop within a loop. The noise displacement in this case is being handled outside the wrangle. Here we only take care of the rings themselves.

```c#
int count = chi("count");
int ringcount = chi("ringcount");

for(int i=0; i <= count; i++)
{
    float radius = ch("radius");
    float angle = radians(i);

    for(int j=0; j<=ringcount; j++){
        float rel =  sin(radius * j);
        vector pos = set(j * cos(angle) * rel, 0, j * sin(angle) * rel)  * ch("size");
        pos.y = 0;
        int pt = addpoint(0, pos);
        setpointattrib(0, "id", pt, j, "set");
    }
}
```

![growth-rings_small](https://user-images.githubusercontent.com/81909946/113510793-c20b9180-955c-11eb-852b-4b178a871feb.gif)

[Download example scene file.](https://github.com/ribponce/particula/blob/25c22918431b8b8b9e271b59f379d85a1f597a35/vex/files/particula_growth-rings_SHARE.hipnc)
