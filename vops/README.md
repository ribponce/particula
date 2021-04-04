Basically, a network for scripting vex visually. A coworker which has been much longer on this road than I am told me that, for my surprise, before you could only interact with vex through VOPs (so called snippets), and that SOP-level wrangles are a much newer addition to Houdini.

For some operations I will prefer VOPs over vex in a heartbeat, such as when I need to create noise patterns and stuff. I’ve seen some insane scripts doing the craziest things, which always reminds me that I have a long road ahead of me. Still, let’s start small, and put into this page some interesting setups that keep popping up to serve as reference later on.

# Procedural Rope

I tried a couple of times to do something similar in the past using vex, only to get frustrated when it didn’t work as expected. Finally gave it a more patient go in VOPs and it came out really nice. Here is an overview of the network handling the curve’s spiraling.

![procedural-rope-vopnet](https://user-images.githubusercontent.com/81909946/113512268-267e1f00-9564-11eb-8cf9-718073f58fd4.png)

It might look more complex than it actually is. It consists basically of using sin() and cos() functions to displace the points. The trick is to properly displace them perpendicularly to their tangents, so no matter how we orient the input curve it will always swirl nicely along.

By the way, there is a simple setup to calculate @tangentu based on the curve’s points positions prior to this VOP. On the provided file you will also find a VOP solution to that, but here’s the snippet in vex.

if(@ptnum!=npoints(0)-1){
v@tangentu = normalize(point(0, "P", @ptnum+1) - @P);
}else{
v@tangentu = normalize(@P - point(0, "P", @ptnum-1));
}

For simulating it we’re using a simple vellum setup. One quick note on that; I found that it’s much easier to simulate the splines in fairly low resolution (in terms of point count), and upres/polywire it later on. It’s important to pump up the constraint iterations on the vellum solver and eventually substeps as well, though it will always depend on what you’re trying to achieve.

*File updated: I implemented a few extra things to the VOP network handling the curve’s intertwining, now supporting taper, squish and also braid style!

![procedural-rope_02](https://user-images.githubusercontent.com/81909946/113512283-38f85880-9564-11eb-90bf-5dc347b4afcf.gif)

[Download scene file.](https://github.com/ribponce/particula/blob/58685e975a3c7e7e5a158f0b19f4ef060d34b410/vops/files/particula_procedural-rope_SHARE.hipnc)

---

# Looping Noise

![looping-noise_01](https://user-images.githubusercontent.com/81909946/113512457-0bf87580-9565-11eb-963f-67877d61c830.gif)
![looping-noise_03](https://user-images.githubusercontent.com/81909946/113512535-609bf080-9565-11eb-9e81-8bc50e2ed737.gif)

Very neat trick I learned after taking a look at Matt Estela’s legendary [odforce thread](https://forums.odforce.net/topic/24056-learning-vex-via-animated-gifs-bees-bombs/), where he mostly as a vex exercise replicates [bees&bombs](https://twitter.com/beesandbombs) awesome gifs in Houdini.

One of the [links shared](https://necessarydisorder.wordpress.com/2017/11/15/drawing-from-noise-and-then-making-animated-loopy-gifs-from-there/) by Matt led me to explore these looping noises. I also tried to implement similar effects using vex only, but the noise functions are just clumsy to work with, in my opinion. I stand by my general rule; if any sort of noise is needed, stick with VOPs.

The important bit of that page is below; a formula for evaluating a 4D noise, using coordinates x and z, as well as sine and cosine functions as arguments.

```
noise.eval(scale*x,scale*y,radius*cos(TWO_PI*t),radius*sin(TWO_PI*t))
```

Never mind that function name; over there it’s all done in Processing. But it isn’t all that hard to replicate the same in Houdini. Here’s a graph overview:

![looping-noise_vopnet](https://user-images.githubusercontent.com/81909946/113512548-6eea0c80-9565-11eb-9825-555f38173a8a.png)

Check out the file attached to play around with it, if you don’t feel like rebuilding it from scratch. The noise that worked best after a lot of experimentation is still Unified Noise’s Perlin Flow, but feel free to explore. Changing the input between a mesh and a point cloud can also greatly alter the look of it.

Download scene file.


