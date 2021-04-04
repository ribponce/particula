# Interactive Python States

[See on Youtube](https://www.youtube.com/watch?v=DTReVsTmKNY)

In this tutorial we take a look at python viewer states inside Houdini 18, and how its practicality can help technical artists develop interactive, easy-to-use tools for fast and smart content creation.

The first iteration of this project was an attempt at replicating by Simon Verstraete’s great results on his article at 80lv. After a few messages he was able to point me in the right direction. So I really appreciate that, thanks Simon!

However, after getting a hang of how things could be implemented, I soon realized that simply extruding geometry wasn’t going to achieve the results I envisioned.!

[![cubes_py_demo01_reduced](https://user-images.githubusercontent.com/81909946/113513914-208c3c00-956c-11eb-835d-8964c1419ac6.gif)](https://www.youtube.com/watch?v=DTReVsTmKNY)

*This is where all started. On the first iteration of this project the only accepted input was a flat grid. It also relied solely on extruding primitives, which later forced me to search for other alternatives.*

The reason for that is also mentioned briefly in the video, but to be more specific, I wanted the little boxes to always obey a master grid, so whenever I performed a click for the creation of an overhang (boxes that are connected only by its horizontal neighbor), its shape would also perfectly match with the column of cells below it.

The way I handled that was to start working with a pre-constructed grid, and any actions performed would simply check for the corresponding cell on the grid and merge it back to the original geometry.

[![cubes_py_demo02_reduced](https://user-images.githubusercontent.com/81909946/113513955-4e718080-956c-11eb-9b3a-0297ff8e4c45.gif)](https://www.youtube.com/watch?v=DTReVsTmKNY)

*Still using an old medieval roof asset I created previously, in here I was already able to achieve the results I envisioned for the cells behavior, but still wasn’t satisfied with the fact that only flat inputs were accepted.*

I believe this method I came up with (thoroughly described in the video) is theoretically very effective, specially after I managed to handle its creation adaptively, however I agree its implementation could have been more technical.

A geometry creation process by demand would have been ideal, and I hope I will be able to implement something more robust in the near future.

Now onto to the Favela Tool (or Slum Tool) also showcased in the video, a few things I’d like to point out. It’s still under development, like I mentioned, and a few things will have to be reworked.

[![favela_demo_01_reduced](https://user-images.githubusercontent.com/81909946/113513982-65b06e00-956c-11eb-9822-b068004f6a59.gif)](https://www.youtube.com/watch?v=DTReVsTmKNY)

*The project took a sharp turn, and I decided to attempt and create a tool that could generate favelas (slums) instead of little medieval buildings, to take full artistic advantage of the newly implemented acceptance of any quad grid as input.*

I spent most of my time developing a couple of procedural roofs that could be applied to any quad input, and that technique seems to be working well. It would still need some research and experimentation If I wanted to make it work with inputs of n-sides, which I something I indeed would like to see.

[![favela_demo_02_reduced](https://user-images.githubusercontent.com/81909946/113513992-7234c680-956c-11eb-8196-4b68fc06e628.gif)](https://www.youtube.com/watch?v=DTReVsTmKNY)

*I was always concerned about performance, and made sure that everything was inside compiled blocks, whenever applicable. From here, I spent time implementing streets, as well as a random generator within the tool, both of which can be seen in video.*

There isn’t currently a procedural system in place for module placement of the walls. They are all simple one-faced textured primitives. So that, as well as some many other minor things, are also on my to-do list going forward.

That’s pretty much it, I appreciate if you read it this far.

Apart from being able to download the .hip file here, the python code for the click-extrude-tool is also pasted below, just for the sake of having it available online without the need of downloading it. Keep in mind that if copied to a new HDA, the parameters used must match or be replaced accordingly.

```python

import hou
import viewerstate.utils as su
from random import seed
from random import sample

class MyState(object):
    def __init__(self, state_name, scene_viewer):
        self.state_name = state_name
        self.scene_viewer = scene_viewer
        self._geometry = None
        self.node = None
               
        prims = []
        pset = []
        undoprims = []
        undopset = []
        iter = 0
        undo = 0
        randiter = 0
        
        self.prims = prims
        self.pset = pset
        self.undoprims = undoprims
        self.undopset = undopset
        self.iter = iter
        self.undo = undo
        self.randiter = randiter
    def SetPrompt(self, kwargs):
        prompt = (
            "1. LMB and drag. Selection will be extruded.\n"
            "2. Ctrl+LMB will undo single extrusions.\n"
        )
        self.scene_viewer.setPromptMessage(prompt)
        
    def UpdateStashedGeo(self, kwargs):
        self.node.parm("stash").set(self.node.node("STASH_GEO").geometry())
	
    def UpdateStashUndo(self, kwargs):
        self.node.parm("stash").set(self.node.node("STASH_UNDO").geometry())    
        
    def UpdateFinalStash(self, kwargs):
        self.node.parm("final_stash").set(self.node.node("STASH_GEO").geometry())    

    def onEnter(self, kwargs):
        node = kwargs["node"]
        self._geometry = node.node("PRE_EXTRUDE").geometry()
        self.node = kwargs["node"]      
        with hou.undos.disabler():
            self.SetPrompt(kwargs)

    def onMouseEvent(self, kwargs):
            
        node = kwargs["node"]
        ui_event = kwargs["ui_event"]
        state_parms = kwargs["state_parms"]
        
        reason = ui_event.reason()
        device = ui_event.device()
        
        ray_origin, ray_dir = ui_event.ray()
        gi = su.GeometryIntersector(self._geometry)
        gi.intersect(ray_origin, ray_dir, snapping=False)

        if reason == hou.uiEventReason.Active:
            if not device.isCtrlKey():
                prim = gi.prim_num
                self.prims.append(prim)
                self.pset = list(set(self.prims))
                node.parm("group").set(str(self.pset))
                self.undo = 0
            else:
                undoprim = gi.prim_num
                self.undoprims.append(undoprim)
                self.undopset = list(set(self.undoprims))
                node.parm("undo").set(str(self.undopset))
                self.undo = 1

        elif reason == hou.uiEventReason.Changed:
                if self.undo == 0:
                    self.UpdateStashedGeo(kwargs)
                    self.UpdateFinalStash(kwargs)
                    node.parm("group").set(str(""))
                    self.iter += 1                    
                    node.parm("iter").set(str(self.iter))
                    del self.prims[:]
                elif self.undo == 1: 
                    self.UpdateStashUndo(kwargs)
                    self.UpdateFinalStash(kwargs)
                    node.parm("undo").set(str(""))
                    self.iter += 1                    
                    node.parm("iter").set(str(self.iter))
                    del self.undoprims[:]
                    self.undo = 0

    def onParmChangeEvent(self, kwargs):
        #React to parameter changes
        node = kwargs["node"]
        parm_name = kwargs['parm_name']
        parm_value = kwargs['parm_value']
        state_parms = kwargs['state_parms']
        
        if parm_name == "reset":
            # Reset 
            del self.prims[:]
            del self.pset[:]
            del self.undoprims[:]
            del self.undopset[:]
            self.iter = 0
            self.randiter = 0
            node.parm("iter").set(int(0))
            node.parm("group").set(str(""))
            node.parm("undo").set(str(""))         
            self.node.parm("stash").set(None)
            self.node.parm("final_stash").set(None)
            self.UpdateFinalStash(kwargs)

        if parm_name == "generate":         
            # Generate Random
            nprims = self.node.evalParm("nprims")
            amount = self.node.evalParm("amount")
            randseed = self.node.evalParm("randseed")
         
            seed(randseed+amount+nprims+self.randiter)
            sequence = [i for i in range(nprims)]
            subset = sample(sequence, amount)
            self.node.parm('rand_result').set(str(subset))
            self.randiter += 1
            self.UpdateFinalStash(kwargs)
            self.UpdateStashedGeo(kwargs)
            self.node.parm('rand_result').set(str(""))
            print(self.randiter)

def createViewerStateTemplate():
    """ Mandatory entry point to create and return the viewer state 
        template to register. """
	
    state_typename = kwargs["type"].definition().sections()["DefaultState"].contents()
    state_label = "Click Extrude Tool"
    state_cat = hou.sopNodeTypeCategory()
	
    template = hou.ViewerStateTemplate(state_typename, state_label, state_cat)
    template.bindFactory(MyState)
    template.bindIcon(kwargs["type"].icon())
	    
    template.bindParameter(hou.parmTemplateType.Button, name="reset", label="Reset")
    template.bindParameter(hou.parmTemplateType.Button, name="generate", label="Generate Random")
	
    return template

```

