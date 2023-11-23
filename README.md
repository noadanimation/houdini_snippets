# Houdini Snippets
Some handy bits and bobs for Houdini users

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
