# Houdini Snippets
Some handy bits and bobs for Houdini users

## Traverse input nodes during playback
I made this script to help create behind the scenes videos of Houdini SOP networks. 

To use: Put this code in a Python SOP node, connect any node (preferably the end of a super complicated node tree 😉) to the first input, then press play.

What it does? It traverses all the input nodes in sequence - showing a new node for each frame of playback. You can pop a turntable camera in and get a nice overview of all the steps along the way to your final masterpiece!

```
import hou

node = hou.pwd()
geo = node.geometry()

def traversenodes(startnode, traversednodes, branchingtraversednodes=[]):
    inputs = startnode.inputs()
    inputs = [x for x in inputs if x is not None]

    traversednodes.append(startnode)
    if len(inputs)==0:
        return traversednodes
    elif len(inputs)==1:
        if inputs[0] in traversednodes or inputs[0] in branchingtraversednodes:
            return traversednodes
        else:
            return traversenodes(inputs[0], traversednodes, branchingtraversednodes)

    branches = []
    localtraversed = branchingtraversednodes.copy()
    for input in inputs:
        if input in traversednodes or input in branchingtraversednodes: continue

        branch = traversenodes(input, [], localtraversed)
        localtraversed += branch
        branches.append(branch)

    for branch in reversed(branches):
        traversednodes += branch

    return traversednodes

geo.clear()
startnode = node.inputs()[0]
nodes = list(dict.fromkeys(reversed(traversenodes(startnode, []))))

startframe = int(hou.playbar.frameRange()[0])
currentframe = hou.intFrame()

offset = currentframe - startframe
if(offset < 0): offset = 0
elif(offset > len(nodes)-1): offset = len(nodes) - 1

currentnode = nodes[offset]

for node in nodes:
    node.setTemplateFlag(False)
    node.setSelectableTemplateFlag(False)
currentnode.setSelectableTemplateFlag(True)
```

## Print SOP verb parameters
When working with the new APEX graphs in Houdini 20, a lot of the "SOP verb" nodes have very opaque parameter menus, and it's hard to tell what settings to use.

Place a temporary version of the same node in the SOP network (not in APEX), set its parameters as you like, then plug it into a Python SOP with this code:
```
node = hou.pwd()
input = node.inputs()[0]

invoke = input.verb()
invoke.loadParmsFromNode(input)
print(f"\n---  Parms for sop-verb {input.type().name()}  ---")
for key, val in invoke.parms().items():
    print(f"{key}: '{val}'")
```
This will print a list of all the parameter names and values, and you can enter these into the APEX parameters.
