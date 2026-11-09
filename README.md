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

### Update November 2026:
Recently I broke through the limit of the above snippet - with a very detailed node network, you'll eventually run into a recursion error, and the code used to traverse the nodes gets very slow when run each frame for lots of nodes. To fix those issues I've rewritten the snippet to avoid the recursion, and split it into two scripts - the first script does the recursion and writes the traversed nodes to a detail attribute, the second script sets the visible node.

To use these, create a Python node for each script, plug your node network into the first Python node (once it's calculated you can freeze the node so it doesn't recalculate every frame), and plug the output of that Python node into the second Python node. The script will cycle the template flag over each node in your network during playback while you're inside the SOP Geometry.

#### Script 1:
```
import hou

node = hou.pwd()
geo = node.geometry()

def filteredinputs(node):
    inputs = node.inputs()
    inputs = [x for x in inputs if x is not None]
    return inputs
    
startnode = node.inputs()[0]

currentlist = [[startnode,0,node]]
pending = [[startnode,0,node]]
visited = [node]
newinputs = True
branchhead = None
branch = []

maxgen = 5000
while len(pending)>0 and maxgen > 0:
    maxgen -= 1
    now = pending.pop(0)
    if branchhead == None:
        branchhead = now[2]
        branch = []
    
    inputs = filteredinputs(now[0])
    
    if not now[0] in visited:
        branch.append(now[0])
    if now[0] in visited or len(inputs)==0:
        index = visited.index(branchhead)
        visited = visited[0:index] + list(reversed(branch)) + visited[index:]
        branchhead = None
        continue
    
    newpending = []
    for i, inputnode in enumerate(inputs):
        if inputnode in visited: continue
        if inputnode in branch: continue
        data = [inputnode, now[1]+1, now[0]]
        if data in newpending: continue
        newpending.append(data)
        
    pending = newpending + pending
       
print("Calculated nodes to visit: " + str(len(visited)))

paths = ""
for v in visited:
    paths = paths + v.path() + ", "

geo.addAttrib(hou.attribType.Global, "nodepaths", "/")
geo.setGlobalAttribValue("nodepaths", paths)
```
#### Script 2
```
import hou

node = hou.pwd()
geo = node.geometry()
       
paths = geo.stringAttribValue("nodepaths").split(",")
visited = []
for p in paths:
    p = p.strip()
    if p == "": continue
    visited.append(node.node(p))
        
print("Visiting " + str(len(visited)) + " nodes")

startframe = int(hou.playbar.frameRange()[0])
currentframe = hou.intFrame()

offset = currentframe - startframe
if(offset < 0): offset = 0
elif(offset > len(visited)-1): offset = len(visited) - 1

currentnode = visited[offset]
if currentnode != node:
    geo.clear()

for node in visited:
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
