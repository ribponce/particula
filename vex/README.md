In this page I will be compiling interesting snippets and examples. I intend to simplify them for the sake of organisation, but there are some cases where I go a little more in depth. I hope I can often provide a scene file along with the examples, though not guaranteed.

I will still often struggle translating ideas into code, but having such a page is also a personal incentive to keep practicing and learning. Therefore, any requests, questions or suggestions are more than welcome.

[SideFX’s VEX Functions Reference](http://www.sidefx.com/docs/houdini/vex/functions/index.html)

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

---

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

---

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

---

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

---

# For Loop and Timeshift

The previous example allows us to freely drape any curve, which we can expand in many different ways. For example, what if we want to copy these curves repetitively and offset their drape amount slightly, plus animate the  whole thing? I learned a similar trick from cgwiki, utilising Timeshift in combination with a For Loop. Here is the overall workflow for such setup:

1. Have an animated piece of geometry that we want to copy and offset. In this particular case it’s a simple sin() function at the amount parameter we created at the wrangle above, taking $F (the current animation frame) as argument.
2. Append a for loop, set it to fetch input, method by count and merge each iteration. Create meta import node. We can now refer to this node to access our loop information.
3. We timeshift the input using the detail attribute from the meta node as argument, added to the current $F. We also transform it in some arbitrary direction right after using the same principle.

In this case we are simply using the iteration number as ‘step’ for both operations, just for the sake of simplicity. We could definitely fit the value to a new desired range, or manipulate it however we need it.

![draping-cables](https://user-images.githubusercontent.com/81909946/113509724-71de0080-9557-11eb-80c9-9553d4af4917.gif)

[Download example scene file.](https://github.com/ribponce/particula/blob/b4aa077ec8c758d0b0d386931ac16a4f25ca3a1f/vex/files/particula_draping-cables_SHARE.hipnc)

---

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

---

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

---

# Scramble color between neighbors

We create an array for each of our incoming points and store their neighbors. Then iterate over them, getting their Cd’s and storing into a new array. Lastly, using the pop() function we get a random item of that same array and assign its color to our current point. I might be mistaken, but I think this does not guarantee that all colors will be exclusive due to its parallelism. The results are nice nonetheless.

```c#
vector nbarray[];
int npts[] = nearpoints(0, @P, ch("maxdist"), chi("maxpts"));
foreach(int i; npts)
{
    vector curCd = point(0, "Cd", i);
    append(nbarray, curCd);
    @Cd = pop(nbarray, (int)fit01(rand(i),0,len(npts)));
}

//v[]@nb_array = nbarray; //debug
```



# Pick random from single array

To make a single array and randomize the colors all over our incoming geometry we need two wrangles. Well, at least that’s how I did it for now. This one run as detail, where we simply append all of the colors from all points into a single array.

```c#
vector cdarray[];
for(int i=0; i<npoints(0); i++)
{
vector curCd = point(0, "Cd", i);
append(cdarray, curCd);
}
v[]@cd_array = cdarray;
```

We can then run over points and have them randomly get an item from that array and use it as their new Cd. Just like the previous example, I believe many items from our array might get repeated or simply ignored. I’ll try to investigate further, new methods that ensure all incoming colors are reused only once. Suggestions are more than welcome.

```c#
vector cdarray[] = detail(0, "cd_array", 0);
vector newCd = pop(cdarray, (int)fit01(rand(@ptnum),0,npoints(0)));
setpointattrib(0, "Cd", @ptnum, newCd, "set");
```

![scramble-between-neighbors-low](https://user-images.githubusercontent.com/81909946/113510844-0b5be100-955d-11eb-9277-eb9e4d28cb25.jpg)

[Download example scene file.](https://github.com/ribponce/particula/blob/1183d64d18313d86d0d01ac843c10397f9f0cd8e/vex/files/particula_scramble-colors_SHARE.hipnc)

---

# Grow attributes

This is basically using the nearpoints() function, which might restrict it in some ways but in the other hand it should run faster than opening point clouds for checking neighbors. Here we are coloring some points red in the original geometry either manually or randomly, and growing their Cd attribute across the surface. Run it in detail mode.

To be honest this one seems overly complicated and doesn’t achieve that much, but I’ll leave it here nonetheless in case I revisit something similar in the future. It was a good vex exercise at least.

```c#
int infected[];
for(int j=0; j<=npoints(0); j++)
{
    vector curCd = point(0, "Cd", j);
    if(curCd.r==1&&curCd.g==0&&curCd.b==0)
    {
        append(infected, j);
    }
}

//i[]@inf = infected; //debug

for(int i=0; i<=len(infected); i++)
{
    int curpt = infected[i];
    vector npos = point(0, "P", curpt);
    int npts[] = nearpoints(0, npos, ch("maxdist"));
    foreach(int pt; npts)
    {
        int nbcd = point(0, "Cd", pt);
        if(nbcd!=0)
        {
            setpointattrib(0, "Cd", pt, {1,0,0}, "set");
        }
    }
}
```

[Download scene file.](https://github.com/ribponce/particula/blob/5a970001dea7829003b08aaf9ffae0d268b5928a/vex/files/particula_grow-attribute_SHARE.hipnc)

---

# Vase generator

A short vex exercise. I’m always fascinated about being able to use ramps to create 3D shapes in Houdini. This is something I used to do quite a lot when I started experimenting with Rhino’s plugin Grasshopper, so I decided to give it a go in vex. Similar to the growth rings snippet, here we also wrap a for loop inside another, where each is responsible for handling the horizontal and vertical sections of our container, respectively.

![vase-gen](https://user-images.githubusercontent.com/81909946/113511045-1ebb7c00-955e-11eb-855e-d9521fb7a206.gif)

Note as well the last two lines of code. We assign two attributes with setpointattrib() that later allows us to connect the points accordingly.

```c#
int h_div = chi("h_div");
for(float i=0; i<h_div; i++)
{
    @P.y = i / h_div;
    float v_div = ch("v_div");
    
    for(float j=0; j<v_div; j++)
    {
        float ramp = chramp("P", @P.y);
        float angle = radians(j) * (360/v_div);
        vector pos = set(cos(angle)*ramp, @P.y * ch("height"), sin(angle)*ramp);
        int pt = addpoint(0, pos);
        setpointattrib(0, "horizontal", pt, i, "set");
        setpointattrib(0, "vertical", pt, j, "set");
    }
}
```

I think there is no need for a scene file on this one. After running the snippet in detail mode, appending an Add node (Polygons>By Group, Add by Attribute), Skin and Polyextrude is all we need.

---

# Cellular Automaton

The patterns that emerge from automatons such as Conway’s Game of Life (amazing wiki entry btw, with more in-depth explanation) have always fascinated me. I decided to give it a go in vex as an exercise. In principle, every point is a “cell” that has its neighbours check at every iteration. If specific conditions are met, the cell’s state is either set to alive or dead (populated or unpopulated, respectively). In principle:

1. Any live cell with fewer than two live neighbours shall die.
2. Any live cell with more than three live neighbours shall die.
3. Any dead cell with exactly three live neighbours comes to life.
4. Any live cell with two or three live neighbours is passed on to the next generation.

Rule number 4. can be ignored, since it’s basically byproduct from other conditions. The code below runs over points inside a solver. Let’s go through it.

```c#
int dead[];
int nearpts[] = nearpoints(0, @P, 1.9);
foreach(int num; nearpts)
{
    if(point(0, "dead", num)==1)
    {
        append(dead, num);
    }
    removevalue(dead, @ptnum);
}
if(len(nearpts)<=6)
{
    i@border=1;
    i@dead=1;
}
if(@border!=1)
{
    if(@dead==0 && len(dead)>=7)
    {
        i@dead=1;
    }
    else if(@dead==0 && len(dead)<5)
    {
        i@dead=1;
    }
    else if(@dead==1 && len(dead)==5)
    {
        i@dead=0;
    }
}
```

Breaking it down; with the nearpoints() function we create an array for each of the cells and iterate over them. We state that if the current num (representing a neighbour cell) being evaluated is dead, we append it to a new array named dead[]. We also use removevalue() to make sure the array consists solely of its neighbours (exclude itself).

Next we check for the border. Life supposedly uses an infinite grid, but for practical reasons we state that border cells are simply dead, so we have our little system contained to an arbitrary size. After some research I found that this is probably the most common way of adapting it.

Then we use the dead[] array we created to specify the state switching of the cells. Since the maximum value of dead neighbours is 8 (remember, we are searching in a radius of 1.9, in a grid where the closest cells are 1 unit apart – vertical, horizontal and diagonal direct neighbours are taken in consideration), we only need to adapt the rules’ syntax slightly. if(@dead==0 && len(dead)>=7)  corresponds to rule 1., “live cell with fewer than two live neighbours”, and so forth.

Check out the scene file below for more details. Really had some fun doing this one!

![cel-automaton-01](https://user-images.githubusercontent.com/81909946/113511112-7c4fc880-955e-11eb-8966-7dca6b47a518.gif)
![cel-automaton-03](https://user-images.githubusercontent.com/81909946/113511115-7e198c00-955e-11eb-9e9d-a967489fe02d.gif)

[Download scene file.](https://github.com/ribponce/particula/blob/a50bcf1edc042273ed146e645fcd035f8ebf4b29/vex/files/particula_cellular_automaton_SHARE.hipnc)

---

# Packed geometry intrinsics

In certain situations we might need to “reset” the rotation and translation of a packed primitive or even an alembic geometry to the origin, and this vex method that uses intrinsic attributes seemed to be really efficient.

This specific technique was really useful when offsetting the noise of a shader properly while the geometry moves in 3D space. Again, I could be wrong and there is a much easier solution to this (probably), but when working with an animated alembic, I found that performing all of your operations on a static geometry at the origin and transforming it back to the original alembic position using Fetch at the object level solved the issue where the noise wouldn’t “stick” to the geometry.

The basic workflow would then be to place your packed geo at the origin with the snippet below, do whatever you need to it and on the upper level fetch its parent position. Also make sure to toggle Use Parent Transform of Fetched Object.

```c#
vector pivot = primintrinsic(0, "pivot", 0);
matrix3 mat = 1;

setprimintrinsic(0, "transform", 0, mat, "set");

matrix abcMat = primintrinsic(0, "packedfulltransform", 0);
vector abcPos = cracktransform(0, 0, 0, pivot, abcMat);

@P = (@P - abcPos);
```

---

# Compute tangent and bitangent

I noticed that sometimes when using Polyframe on a closed curve I would get weird tangents and bitangents on that geometry’s first point, so I started studying how to mimic it and fix that little issue in vex. I did the first sketch of it in vops (which really helps organizing everything), and finally migrated it to text due to a considerable faster cook time (faster than polyframe as well).

This one has a very neat trick that I really like; using modulo to procedurally get the next or previous point number. There’s also a simple check about whether its an open or closed geometry using the neighbours() function, and another check to see if the geometry is a curve or not by comparing the vertex count to the point count.

I used the SideFX’s documentation on the Polyframe node to understand how the tangent and bittangent are computed. This specific snippet is using a Two Edges computation style, which basically means their tangents are computed through the smoothed difference between a point’s position and their neighbors’, but also will vary depending whether the geometry is a closed shape, a closed curve, or an open curve.

This runs in detail mode, and requires a simple pointwrangle before it where we make sure the normal attribute is initialized (@N = @N;).

```c#
//declare variables
vector t, tc, bt, btn, btnc;
int nbs[];
int closed, curve;
//check if geometry is a curve
int numvtx = primvertexcount(0, 0);
if(numvtx>@numpt){
curve=1;}
//iterate over points
for(int i=0; i<=@numpt; i++){
//fetch point info
    int nextpt = (i+1)%@numpt;
    int prevpt = (i-1)%@numpt;
    vector curN = point(0,"N",i);
    vector curpos = point(0,"P",i);
    vector nextpos = point(0,"P",nextpt);
    vector prevpos = point(0,"P",prevpt);
//check if geometry is open or closed
    if(i==0 || i==@numpt-1){
        nbs = neighbours(0, i);
        foreach(int nb; nbs){
            if(nb==0 || nb==@numpt-1){
            closed = 1;}
        }
    }
//general tangent and bitangent    
    t = normalize(lerp(curpos-nextpos, prevpos-curpos, 0.5));
    bt = cross(curN,t);
    btn = cross(t,curN);
//set them according to the geometry
    if(curve==0){
        if(closed==1){
            setpointattrib(0,"tangentu",i,bt,"set");
            setpointattrib(0,"tangentv",i,t,"set");
        }else{
            if(prevpt==@numpt-1){
            tc = normalize(curpos-nextpos);}
            else if(nextpt==0){
            tc = normalize(prevpos-curpos);}
            else{
            tc = normalize(lerp(curpos-nextpos, prevpos-curpos, 0.5));}
            btnc = cross(tc,curN);
            setpointattrib(0,"tangentu",i,tc,"set");
            setpointattrib(0,"tangentv",i,btnc,"set");
        }
    }else{        
        if(closed==1){
            setpointattrib(0,"tangentu",i,t,"set");
            setpointattrib(0,"tangentv",i,btn,"set");
        }else{
            setpointattrib(0,"tangentu",i,bt,"set");
            setpointattrib(0,"tangentv",i,t,"set");
        }
    }
}
```

---

# The Trapped Knight

Inspired by a ![Numberphile video](https://www.youtube.com/watch?v=RGQe8waGJ4w), this small exercise was interesting to implement in vex. Since it’s an iterative process, I knew we would need a solver to make it happen.

![trapped_knight](https://user-images.githubusercontent.com/81909946/113511301-76a6b280-955f-11eb-9935-7ba4944b9de1.gif)

The first part is to generate a squared spiral, and a lot of help on the topic came from this SO thread. The Vex code to accomplish it is shown below. It runs in detail mode, and creates just the points for the solver to act on.

```c#
float X = chi("count");
float Z = chi("count");
int x = 0; int z = 0; int dx = 0; int dz = -1;
vector pos;
int t = (int)max(X,Z);
int maxiter = t*t;
for(int i=0; i<maxiter; i++){
    if((-X/2 <= x) && (x <= X/2) && (-Z/2 <= z) && (z <= Z/2)){
        pos = set(x,0,z);
        addpoint(0,pos);
    }
    if( (x == z) || ((x < 0) && (x == -z)) || ((x > 0) && (x == 1-z))){
        t = dx;
        dx = -dz;
        dz = t;
    }
    x += dx;
    z += dz;
}
```

The way I approached it was fairly simple: At every iteration of the loop we have a single point being considered as the selected one. From that point position we know that it can only move in a “L” shape, which can easily be coded as an array of coordinates, each representing the end position of an “L” shaped move (there are 8 in total).

```c#
vector coords[] = { {2,0,-1} , {1,0,-2} , {-1,0,-2}, {-2,0,-1}, {-2,0,1}, {-1,0,2}, {1,0,2}, {2,0,1} };
```

At every step, we look up all points on those coordinates and store them inside a new array. By sorting them in increasing order, we make sure the first element of the array is the lowest possible value for the knight to visit. We then make that option the selected point instead, and mark the previous one as already visited as well, since the knight is not allowed to revisit any positions.

Take a look at the scene file below for more details on how to accomplish it.

[Download scene file.](https://github.com/ribponce/particula/blob/607c1e7d27b18f404c0cf23a08000ca164207e0d/vex/files/particula_trapped-knight_SHARE.hipnc)

---

# Custom FtoA function (Convert Float to String)

This function exists as HScript, but not in vex. Here's a simple snippet to convert floats to string. Alter the code to get more decimals.

```c#
string ftoa(float f){
    // inspired by
    // https://fsimerey.com/index.php/memo/33-houdini-vex-convert-float-to-string-with-2-decimals
    string trunc = itoa(int(trunc(f)));
    int ifrac = int(rint(frac(f)*100));
    string frac = "00";
    if (ifrac == 100) {
        trunc = itoa(int(trunc(f)) +1); 
    }
    else {
        frac = sprintf("%01d", ifrac); // How many decimals?
    }
    string s = trunc + "." + frac;
    return s;
}
```

