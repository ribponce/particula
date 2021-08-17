# Houdini Morse Machine

[![morse-machine_demo_01](https://user-images.githubusercontent.com/81909946/129100191-be036bd3-8b3a-4b04-a7d7-19d9722e4448.gif)](https://www.youtube.com/watch?v=rKTLy-FtEzE)

[Watch on Youtube](https://youtu.be/3YpBgvkojLM)

In this video I walk you through a setup which converts any text into Morse code. Additionally, it will also generate the audio accordingly. Very interesting bits here, and it was particularly a challenge to learn a bit more how CHOPs work.

This project started off as something somewhat unrelated. I was making a simple looped animation where dots and dashes would scroll around and subtly form words, however that whole thing got me thinking how could I actually design a system that would allow me to modify the words and not rely so much on keyframing things. And here we are. I ended up creating something that turned out to be much more intrinsically interesting that what I was developing in the first place, in my opinion.

Huge shoutout to [Robin Bonhoure](https://twitter.com/rbonhoure) who was more than kind to provide a solution to my first version of the setup in Python. Inspired by him I was able to rework the whole thing and do it my own way in vex.
Below I'm pasting the code as seen in the video and in the .hip file, just for ease of access. This takes care of the linear decoding.

```c#
// Inputs
string input = chs("input");
int letter_spacing = chi("letter_spacing");
int word_spacing = chi("word_spacing");
int line_break = chi("line_break");
float line_spacing = chf("line_spacing");

// Morse Dictionary
dict morse = set( "A",".-", "B","-...",
                    "C","-.-.", "D","-..", "E",".",
                    "F","..-.", "G","--.", "H","....",
                    "I","..", "J",".---", "K","-.-",
                    "L",".-..", "M","--", "N","-.",
                    "O","---", "P",".--.", "Q","--.-",
                    "R",".-.", "S","...", "T","-",
                    "U","..-", "V","...-", "W",".--",
                    "X","-..-", "Y","-.--", "Z","--..",
                    "1",".----", "2","..---", "3","...--",
                    "4","....-", "5",".....", "6","-....",
                    "7","--...", "8","---..", "9","----.",
                    "0","-----", ", ","--..--", ".",".-.-.-",
                    "?","..--..", "/","-..-.", "-","-....-",
                    "(","-.--.", ")","-.--.-");
                    
// Define Functions
int add_point(vector pos; int m; string text; int word; int last){
    int pt = addpoint(0, pos);
    setpointattrib(0, "m", pt, m, "set");
    setpointattrib(0, "text", pt, text, "set");
    setpointattrib(0, "word", pt, word, "set");
    setpointattrib(0, "last", pt, last, "set");
    return pt;
}

// Declare attributes
string words[] = split(input, " ");
string keys[] = keys(morse);
vector pos = {0,0,0};
int word_index = -1;

// Main
foreach(string word; words){
    word_index++;
    foreach(string text; word){
        string code = morse[toupper(text)];
        int len = len(code);
        int count_c = 0;
        foreach(string c; code){
            count_c++;
            
            int m = -1;
            if(c == "."){ m = 1;}
            else{ m = 2;}
            
            int last = 0;
            if(count_c == len){
                last = 1;
            }
            
            if(line_break){
                pos.z = word_index * line_spacing;
            }

            add_point(pos, m, text, word_index, last);
            pos.x += (letter_spacing * m) + (letter_spacing * last);
            
        }
    }
    // This section is for adding spacing data points between words
    // Runs after every word loop
    add_point(pos, -1, "", -1, 1);
    if(line_break){
        pos.x = 0;
    }else{
        pos.x += word_spacing;
    }
}
```

I mentioned a Auto Hotkey script in the video that allows us to interactively type text and have the corresponding Morse code generated on the fly. That script is to be found in the files directory, alongside with the houdini file and the necessary mp3 files for the audio generation.

Thank you very much, and I will see you on the next one!

[Download the .hip file here](https://github.com/ribponce/particula/blob/master/tutorials/houdini_morse_machine/files/particula_morse_machine_SHARE.hip)




