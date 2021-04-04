Basically, a network for scripting vex visually. A coworker which has been much longer on this road than I am told me that, for my surprise, before you could only interact with vex through VOPs (so called snippets), and that SOP-level wrangles are a much newer addition to Houdini.

For some operations I will prefer VOPs over vex in a heartbeat, such as when I need to create noise patterns and stuff. I’ve seen some insane scripts doing the craziest things, which always reminds me that I have a long road ahead of me. Still, let’s start small, and put into this page some interesting setups that keep popping up to serve as reference later on.

# Procedural Rope

I tried a couple of times to do something similar in the past using vex, only to get frustrated when it didn’t work as expected. Finally gave it a more patient go in VOPs and it came out really nice. Here is an overview of the network handling the curve’s spiraling.

It might look more complex than it actually is. It consists basically of using sin() and cos() functions to displace the points. The trick is to properly displace them perpendicularly to their tangents, so no matter how we orient the input curve it will always swirl nicely along.

By the way, there is a simple setup to calculate @tangentu based on the curve’s points positions prior to this VOP. On the provided file you will also find a VOP solution to that, but here’s the snippet in vex.

if(@ptnum!=npoints(0)-1){
v@tangentu = normalize(point(0, "P", @ptnum+1) - @P);
}else{
v@tangentu = normalize(@P - point(0, "P", @ptnum-1));
}

For simulating it we’re using a simple vellum setup. One quick note on that; I found that it’s much easier to simulate the splines in fairly low resolution (in terms of point count), and upres/polywire it later on. It’s important to pump up the constraint iterations on the vellum solver and eventually substeps as well, though it will always depend on what you’re trying to achieve.

*File updated: I implemented a few extra things to the VOP network handling the curve’s intertwining, now supporting taper, squish and also braid style!
