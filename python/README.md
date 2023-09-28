# Manual cook shortcut

Can’t live without this. Create a python script to toggle between manual and auto-update. First, create a custom shelf for yourself if you haven’t yet and add a new tool to it. Name it something simple and paste the code below to the script tab. Alt+Shift+Ctrl+LMB on the shelf tool will prompt you to assign it to a keyboard shortcut. I personally like having it on F5.

```python
if hou.ui.updateMode() == hou.updateMode.AutoUpdate:
    hou.ui.setUpdateMode(hou.updateMode.Manual)
else:
    hou.ui.setUpdateMode(hou.updateMode.AutoUpdate)
```

---


# Working with attributes

Whenever I need to work with a Python SOP it's often the case I need to either read or write (or both) geometry attributes, be it at the point, primitivite or detail level. Python scripting reference page: https://www.sidefx.com/docs/houdini/hom/hou/Geometry.html
Also relevant: https://www.sidefx.com/docs/houdini/hom/pythonsop.html
Here's a simple example I wrote to quickly remind me of some useful snippets.

```python
geo = hou.pwd().geometry()

""" First we create some geometry (not essential for detail attributes) """
prim = geo.createPolygon()
point = geo.createPoint()
point.setPosition((0,0,0))
prim.addVertex(point)

""" Adding and Writing Attributes is mostly straightforward.
For the sake of simplicity we're creating attribute with the same name for every attribute type."""

# Integer
intAttrib = 1337
geo.addAttrib(hou.attribType.Global, "intAttrib", intAttrib, create_local_variable=False)
geo.addAttrib(hou.attribType.Point, "intAttrib", intAttrib, create_local_variable=False)
geo.addAttrib(hou.attribType.Prim, "intAttrib", intAttrib, create_local_variable=False)

# Float
floatAttrib = 97.651
geo.addAttrib(hou.attribType.Global, "floatAttrib", floatAttrib, create_local_variable=False)
geo.addAttrib(hou.attribType.Point, "floatAttrib", floatAttrib, create_local_variable=False)
geo.addAttrib(hou.attribType.Prim, "floatAttrib", floatAttrib, create_local_variable=False)

# String and Array attributes must be added and subsequently set in two steps (not sure why)
stringAttrib = "Hello"
geo.addAttrib(hou.attribType.Global, "stringAttrib", "", create_local_variable=False)
geo.setGlobalAttribValue("stringAttrib", stringAttrib)
geo.addAttrib(hou.attribType.Point, "stringAttrib", "", create_local_variable=False)
point.setAttribValue("stringAttrib", stringAttrib)
geo.addAttrib(hou.attribType.Prim, "stringAttrib", "", create_local_variable=False)
prim.setAttribValue("stringAttrib", stringAttrib)

# Integer Array
intArrayAttrib = [0,1,2,3]
geo.addArrayAttrib(hou.attribType.Global, "intArrayAttrib", hou.attribData.Int, tuple_size=1)
geo.setGlobalAttribValue("intArrayAttrib", intArrayAttrib)
geo.addArrayAttrib(hou.attribType.Point, "intArrayAttrib", hou.attribData.Int, tuple_size=1)
point.setAttribValue("intArrayAttrib", intArrayAttrib)
geo.addArrayAttrib(hou.attribType.Prim, "intArrayAttrib", hou.attribData.Int, tuple_size=1)
prim.setAttribValue("intArrayAttrib", intArrayAttrib)

# Float Array
floatArrayAttrib = [0.5,2.3,5.7]
geo.addArrayAttrib(hou.attribType.Global, "floatArrayAttrib", hou.attribData.Float, tuple_size=1)
geo.setGlobalAttribValue("floatArrayAttrib", floatArrayAttrib)
geo.addArrayAttrib(hou.attribType.Point, "floatArrayAttrib", hou.attribData.Float, tuple_size=1)
point.setAttribValue("floatArrayAttrib", floatArrayAttrib)
geo.addArrayAttrib(hou.attribType.Prim, "floatArrayAttrib", hou.attribData.Float, tuple_size=1)
prim.setAttribValue("floatArrayAttrib", floatArrayAttrib)

# A Vector type doesn't actually exist. We set it to a 3-dimensional float
vectorArrayAttrib = [1.0,-0.7,1.6]
geo.addArrayAttrib(hou.attribType.Global, "vectorArrayAttrib", hou.attribData.Float, tuple_size=3)
geo.setGlobalAttribValue("vectorArrayAttrib", vectorArrayAttrib)
geo.addArrayAttrib(hou.attribType.Point, "vectorArrayAttrib", hou.attribData.Float, tuple_size=3)
point.setAttribValue("vectorArrayAttrib", vectorArrayAttrib)
geo.addArrayAttrib(hou.attribType.Prim, "vectorArrayAttrib", hou.attribData.Float, tuple_size=3)
prim.setAttribValue("vectorArrayAttrib", vectorArrayAttrib)

# String Array
stringArrayAttrib = ["Hello","World"]
geo.addArrayAttrib(hou.attribType.Global, "stringArrayAttrib", hou.attribData.String, tuple_size=1)
geo.setGlobalAttribValue("stringArrayAttrib", stringArrayAttrib)
geo.addArrayAttrib(hou.attribType.Point, "stringArrayAttrib", hou.attribData.String, tuple_size=1)
point.setAttribValue("stringArrayAttrib", stringArrayAttrib)
geo.addArrayAttrib(hou.attribType.Prim, "stringArrayAttrib", hou.attribData.String, tuple_size=1)
prim.setAttribValue("stringArrayAttrib", stringArrayAttrib)

""" Reading Attributes is much less convoluted. We can simply read any attribute directly.
No need to specify attribute type. Here we're storing them on variables.
Note we are referencing [point] and [prim] created on the top.
To get a reference of existing point, for example, we could use the iterPoints() function:
point = geo.iterPoints()[0]  // Where the [0] specifies point number 0. """

intAttrib = point.attribValue("intAttrib")
floatAttrib = prim.attribValue("floatAttrib")
stringAttrib = geo.attribValue("stringAttrib")

# print(intAttrib, floatAttrib, stringAttrib) # Debug
```

---

# Force Cook Callback

Sometimes it's useful to have a button on the upper-level of an HDA that forces a node deep inside a network to be recooked, such as a File SOP, or a Python SOP which reads from data from disk and are unaware of any external changes.

```python
# Here we can target a specific node
hou.node('../PATH/TO/NODE').cook(force=True)

# Alternatively we can force the current node to recook as well
hou.node('./').cook(force=True)
```

The same code can be used at the end of a Python SOP to trigger other nodes in the network to be recooked, allowing us to create chained actions and complex behaviours very efficiently.

---

# Run Python scripts with a button press

Learned this one from Matt Estela, and it's been a life saver. The Python SOP in Houdini behaves like an Attribute Wrangle, and cooks automatically everytime upstream data is changed. Usually that is fine, but in some specific cases you might want end a node tree with some python script, but only execute the code with a button trigger. The steps for that are quite simple:
1. In a Null SOP, Edit Parameter Interface.
2. Add a Multi-Line String Parameter with Language set to Python, name it "code".
3. Add a Button with the following Callback Script:

```python
exec(kwargs['node'].parm('code').eval())
```

This will make the Parameter named "code" evaluate when the button is pressed.

