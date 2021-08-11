# Houdini Morse Machine

![morse-machine_demo_01](https://user-images.githubusercontent.com/81909946/129100191-be036bd3-8b3a-4b04-a7d7-19d9722e4448.gif)

[Watch on Youtube](https://youtu.be/rKTLy-FtEzE)

In this video I walk you through a setup which converts any text into Morse code. Additionally, it will also generate the audio accordingly. Very interesting bits here, and it was particularly a challenge to learn a bit more how CHOPs work.

One interesting bit that was not mentioned in the video is seen below:

![image](https://user-images.githubusercontent.com/81909946/129100396-ee6f4b57-e8cc-45ad-9a65-aee6517f2c9b.png)

In order to procedurally define the correct speed at which the linear morse code should be scrolled to the left, we need to somehow know how long the final generated audio is in frames. First we use the "chopl" expression to get the lenght in samples. Next we need the expression "chopr" to get it in seconds. We can then finally multiply by my scenes frames per second (in this case 24).



