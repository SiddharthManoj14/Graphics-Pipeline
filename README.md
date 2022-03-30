# Graphics-Pipeline
Demo Graphics Pipeline for the Headset

## Background

> ### Review chapters 6, 7 and sections 8.1-8.2 of _Fundamentals of Computer Graphics (4th Edition)_.

### Read Sections 11.4-11.5 and Chapter 17  of _Fundamentals of Computer Graphics (4th Edition)_.

### GLSL

Most of our work will be implemented using the [OpenGL shading language 
(glsl)](https://en.wikipedia.org/wiki/OpenGL_Shading_Language). In many ways,
GLSL code looks like C++ code. However, there are many builtin linear algebra
types (e.g., `vec3` is a 3D-vector type) and geometric functions (e.g.,
`dot(a,b)` computes the [dot product]
Since vectors are often used to represent spatial
coordinates _or_ colors. We can index the coordinates of a vector (`vec3 a`)
using `a.r`, `a.g`, `a.b` or `a.x`, `a.y`, `a.z`. When working with [perspective
projection](https://en.wikipedia.org/wiki/3D_projection#Perspective_projection)
it's often useful to employ 4D [homogeneous
coordinates](https://en.wikipedia.org/wiki/Homogeneous_coordinates) vectors:
`vec4` in glsl. Glsl has many builtin ways to work with differently sized
vectors and matrices. For example, if we have `vec4 h` then we can write `vec3 p
= h.xyz;` to grab the first three coordinates. Similarly, we could write: `vec4
h = vec4(p,1.0)` to convert a 3D Cartesian point to a 4D homogeneous point.

Fortunately, there are many online resources and googling a GLSL-related
question often returns helpful answers.
