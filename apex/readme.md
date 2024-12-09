# Tests and samples using Houdini 20's new APEX graphs

## Bendy limb
A bendy limb setup with IK and extra controls to fine tune the shape of the limb. Work in progress, there are some funky twists.

## Nurbs curve from xform array
An APEX node to create a nurbs curve, and align transforms along the curve. 
### Input 
- a Matrix4 array for the control points
- a float array for the desired output transform positions (in 'u' along the nurbs spline)
### Output
- the nurbs curve geo
- a Matrix4 array for points along the curve - one transform each to match the input 'u' float array
