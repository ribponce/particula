# Houdini Morse Machine

[![morse-machine_demo_01](https://user-images.githubusercontent.com/81909946/129100191-be036bd3-8b3a-4b04-a7d7-19d9722e4448.gif)](https://www.youtube.com/watch?v=rKTLy-FtEzE)

[Watch on Youtube](https://youtu.be/rKTLy-FtEzE)

In this video I walk you through a setup which converts any text into Morse code. Additionally, it will also generate the audio accordingly. Very interesting bits here, and it was particularly a challenge to learn a bit more how CHOPs work.

This project started off as something somewhat unrelated. I was making a simple looped animation where dots and dashes would scroll around and subtly form words, however that whole thing got me thinking how could I actually design a system that would allow me to modify the words and not rely so much on keyframing things.

And here we are. I ended up creating something that turned out to be much more intrinsically interesting that what I was developing in the first place, in my opinion.

I mentioned a Auto Hotkey script in the video that allows us to interactively type text and have the corresponding Morse code generated on the fly. That script is to be found in the files directory, alongside with the houdini file and the necessary mp3 files for the audio generation.

One other interesting bit that was not mentioned in the video is seen below:

![image](https://user-images.githubusercontent.com/81909946/129100396-ee6f4b57-e8cc-45ad-9a65-aee6517f2c9b.png)

In order to procedurally define the correct speed at which the linear morse code should be scrolled to the left, we need to somehow know how long the final generated audio is in frames. First we use the "chopl" expression to get the lenght in samples. Next we need to divide it by it's lenght in seconds, using the expression "chopr". We can then multiply it by the scene fps (in this case 24). Shout out to @wed at the Think Procedural discord channel, who helped me figure this out!

Lastly, in order to actually scroll it at the right speed, we divide the current frame ($F) by the final frame number over the data points' size in the X dimension in world space:

![image](https://user-images.githubusercontent.com/81909946/129101338-6c46bdae-ad7c-42b5-91b7-6b56ef90b5e6.png)

Thank you very much, and I will see you on the next one!

[Download the .hip file here](https://github.com/ribponce/particula/blob/master/tutorials/houdini_morse_machine/files/particula_morse_machine_SHARE.hip)




