# Procedural Scatter

[![particula_procedural-scatter_cover-1024x576](https://user-images.githubusercontent.com/81909946/113514847-2f292200-9571-11eb-80d4-8e4892ac6b26.jpg)](#)

This write-up is divided into two parts. First we take a look at how to export multiple pieces of geometry most efficiently as sequence files. Later we go through building a small network that procedurally checks for inputs on disk and randomly pick items.

## Batch Export Geometry as Sequence

Let’s suppose we have a bunch of packed primitives that we’d like to export as separate geometry files, for example pieces of a fractured rigid body simulation.

![proc-scatter_02](https://user-images.githubusercontent.com/81909946/113514861-3fd99800-9571-11eb-8302-f1277617e4e2.gif)

A great method for achieving that is to use a simple attribute that corresponds to their packed primitive number (Connectivity SOP creates a an integer @class attribute for you that does just that) to selectively link them to the current frame.

In other words, at every frame we keep only one of the packed primitives in the scene (blast node with the expression @class==`$F-1` ) and export them all as a sequence. During that process, we can also reset their position to the origin, and even create proper name attributes to help us organize the files.

![proc-scatter_04](https://user-images.githubusercontent.com/81909946/113514878-508a0e00-9571-11eb-9995-f28e76253e0a.gif)

The snippet below is a very handy method for embedding the @class attribute onto a new @name attribute, using the sprintf function. I use here “model_” as a prefix. Note: adding one unit to @class prevents model names from starting at 0000.

```c#
@class += 1; //optional: models starting from 0001
string model = sprintf('%04d', @class);
s@name = "model_" + s@name + model;
```

We can use functions inside backticks at any given moment for embedding attributes to a file name or path. For example:

**particula_proceduralBoulders_`prims(opinputpath(“.”,0),0,”name”)`.bgeo.sc**

would yield:

**particula_proceduralBoulders_model_0001.bgeo.sc**

given that geometry had a name attribute at the frame of the export (note it’s prims, meaning it should be a string attribute). It’s a very nice technique, and we can even create meaningful folder structures with this. It all depends what we need, and how we wish to organize our files.

![proc-scatter_05](https://user-images.githubusercontent.com/81909946/113514903-77484480-9571-11eb-8ad2-dd7760597c8b.gif)

Middle clicking any parameter will show us the actual path, very handy for making sure there aren’t any typos and the attributes are being properly embed to the file name or path.

## The Scatter Tool

So now we’ve got a folder full of pieces of geometry and we want to either scatter them randomly or use a paint method to bring in random models to our scene. But what if tomorrow we decide to add more models to our candidates pool?

![proc-scatter_06](https://user-images.githubusercontent.com/81909946/113514913-84653380-9571-11eb-8705-8559b0991222.gif)
![proc-scatter_004](https://user-images.githubusercontent.com/81909946/113514915-85966080-9571-11eb-998f-c9fb80129ddf.gif)

In a nutshell, we want to create a small tool that takes in an input folder containing geometry exported as a sequence – just like we did on the previous section -, verifies how many files are there, and randomly picks one of the files from disk every time a new target point is evaluated.

We start by laying down a File SOP, referencing the source folder to get the geometry sequence files from.
Append an Attribute Wrangle, run in detail mode. I couldn’t find an alternative to the opinputpath parameter expression in vex, so in here I’m creating a string called nodepath that will work as our spare input *(`opinputpath(“.”,0)`)*

```c#
// Get file path from first input
string nodepath = chs("nodepath");
// Create attributes
s@file = concat(nodepath, "/file");
s@inputpath = ch(s@file);
```

Next we need to count the files inside our source folder. This is crucial because later we can just add more models to our pool and they will be automatically included as a possible candidate. For that we’re going to use a Python SOP.

```c#
node = hou.pwd()
geo = node.geometry()
# Import modules #
import os, math
# Import input path #
inputpath = geo.stringAttribValue("inputpath")
# Prepare paths #
patharray = inputpath.split("/")
folder = "/".join(patharray[:-1])
# Count files while ignoring hidden #
count = 0
for i in os.listdir(folder):
    if not i.startswith('.'):
        count += 1
# Turn it into a string and set detail attribute #
count = str(count)
geo.addAttrib(hou.attribType.Global, "filecount", "")
geo.setGlobalAttribValue("filecount", count)
```

After that, we should *Timeshift* it just to freeze at the starting frame. The result of our Python SOP after freezing will be the second input on a new Wrangle. The first input here are our target scattered points. The objective of this snippet is to set a random index to the points which is inside the range of our source. Run over points.

```c#
// Create a random seed channel
int randseed = chi("randseed");
// Read the attributes we created
int filecount = atoi(detail(1, "filecount", 0));
setdetailattrib(0, "filecount", filecount, "set");
// Set random index
i@index = (int)rint(fit01(rand(@ptnum, randseed), 0, filecount));
```

The result of that will then be piped into a for-each loop, running over points. Here we will finally be copying a random files from our source folder to our target points.

We start by laying down a File SOP to actually load the geometry. The Geometry File field here should point to the source folder. As a matter of fact, both this and the first File SOP we used to count the source geometry should always be the same, so we will make sure to link them to a string parameter on the upper level, after packing it into an HDA or simply a sub-network. We also need to add a spare input to it, referencing the foreach_begin of our loop. The load method used will be Packed Disk Sequence, and below are the expressions we need to use.

The Frame range goes from one to **detail(-1, “filecount”, 0)**

Similarly, the Sequence index should reference to the index attribute we created right before the loop; **point(-1, 0, “index”, 0)**

Finally, this is connected to a CopyToPoints node, using the input of the For-each loop as the target points.

Here’s an overview of the scatter tool network:

![proc-scatter_tool_01](https://user-images.githubusercontent.com/81909946/113514949-cbebbf80-9571-11eb-9e29-ebd044b1b57e.jpg)

And as usual, the scene file is available below. The whole concept can be expanded to work procedurally and seamlessly with heightfields, which is a great thing.

I realize this sort of tool might become kind of obsolete when Houdini 18 with the new LOPs context hits later in 2019, but nonetheless I think there are small bits in here that are worth sharing.

Thanks for reading, see you in the next one.

Download scene file.

