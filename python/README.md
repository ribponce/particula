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



```python
geo = hou.pwd().geometry()

# Write Detail Attributes
# Integer
intAttrib = 12
geo.addAttrib(hou.attribType.Global, "intAttrib", intAttrib)

# Float
floatAttrib = 5.6
geo.addAttrib(hou.attribType.Global, "floatAttrib", floatAttrib)

# String
stringAttrib = "Hello"
geo.addAttrib(hou.attribType.Global, "stringAttrib", "")
geo.setGlobalAttribValue("stringAttrib", stringAttrib)

# Integer Array
intArrayAttrib = [0,1,2,3]
geo.addArrayAttrib(hou.attribType.Global, "intArrayAttrib", hou.attribData.Int, 1)
geo.setGlobalAttribValue("intArrayAttrib", intArrayAttrib)

# Float Array
floatArrayAttrib = [0.5,2.3,5.7]
geo.addArrayAttrib(hou.attribType.Global, "floatArrayAttrib", hou.attribData.Float, 1)
geo.setGlobalAttribValue("floatArrayAttrib", floatArrayAttrib)

# A Vector type doesn't actually exist. We set it to a 3-dimensional float
vectorArrayAttrib = [1.0,-0.7,1.6]
geo.addArrayAttrib(hou.attribType.Global, "vectorArrayAttrib", hou.attribData.Float, 3)
geo.setGlobalAttribValue("vectorArrayAttrib", vectorArrayAttrib)
```
