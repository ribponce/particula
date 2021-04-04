# Manual cook shortcut

Can’t live without this. Create a python script to toggle between manual and auto-update. First, create a custom shelf for yourself if you haven’t yet and add a new tool to it. Name it something simple and paste the code below to the script tab. Alt+Shift+Ctrl+LMB on the shelf tool will prompt you to assign it to a keyboard shortcut. I personally like having it on F5.

```python
if hou.ui.updateMode() == hou.updateMode.AutoUpdate:
    hou.ui.setUpdateMode(hou.updateMode.Manual)
else:
    hou.ui.setUpdateMode(hou.updateMode.AutoUpdate)
```