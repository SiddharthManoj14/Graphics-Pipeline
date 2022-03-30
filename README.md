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

### On the CPU side

The shader codes run on the GPU.
Let's breifly describe what's happening on the CPU
A pseudo-code version of `main.cpp` might look like:

```
main()
  initialize window
  copy mesh vertex positions V and face indices F to GPU
  while window is open
    if shaders have not been compiled or files have changed 
      compile shaders and send to GPU
    send "uniform" data to GPU
    set all pixels to background color
    tell GPU to draw mesh
    sleep a few milliseconds
```


#### Window

Creating a window is clearly something that will depend on the operating system
(e.g., Mac OS X, Linux, Windows). For this we are using a toolkit called GLFW running on Visual Studio. It works on all major operating
systems. Once the window is open we have access to its contents as an RGB image.
The job of our programs are to fill in all the pixels of this image with colors.
The windowing toolkit also handles interactions with the mouse and keyboard as
well as window resizing.

#### Shader compilation

Unlike C++ code, shaders are compiled _at runtime_. This has a nice
advantage that you can change your shaders without restarting the main program.
So long as the change is noticed and the shaders are recompiled, the rendering
will be immediately updated.

Compilation errors (usually syntax errors) will be output from the main program
and the window will turn black (background color). For example, if your
_fragment shader_ contained:

```
#version 410 core
in vec3 pos_fs_in;
out vec3 color;
void main()
{
  color = pos_fs_in.x;
}
```
#### Data on the GPU

From the perspective of the shader pipeline, data on the GPU is separated into
different types. For example, when we send the mesh vertex positions to the GPU
we're associating one 3D position _per vertex_. This data is considered an
"attribute" of each vertex. Each vertex invokes a single execution of the vertex
shader and its corresponding position is given as the `in vec3 pos_vs_in;`
variable. The output of the vertex shader will be "varying" depending on the
input and computation conducted. Your program is responsible for setting this
output value in `out vec3 pos_cs_in`. This variable is named ending with
`_cs_in` because it will in turn be used as input to the next shader in the
pipeline _tessellation control shader_.

Small amounts of data that is constant and independent of the particular
vertex/tessellation patch/fragment being processed is labeled as "uniform" data.
The prototypical example of this is the perspective projection matrix: `uniform
mat4 proj`. Uniform data is usually changed once per draw frame (e.g., `uniform
float time_since_start` is updated with the number of seconds since the start of
the program) or once per "object" (e.g., `uniform bool is_moon;` is set based on
whether we're drawing the first or second object in our scene).

Large amounts of data that may be randomly accessed by shaders is stored in
"texture" memory (e.g., color texture images). This data must be accessed by
sampling specific pixel values based on a given 2D locations (e.g., the U.V.
mapping of a fragment).

### Tessellation Control Shader

The tessellation control shader determines how to subdivide each input "patch"
(i.e., triangle). Unlike the subdivision we saw with [subdivision
surfaces](https://en.wikipedia.org/wiki/Subdivision_surface), the subdivision is
determined independently for each triangle and _not_ called recursively.
The exact pattern of the resulting triangulation is left largely to
implementation. As the shader programmer, you have control over:

  - the number of new edges each input each should split into
    (`gl_TessLevelOuter[1] = 5` means the edge across from vertex `1` (i.e., the
    edge between vertices `0` and `2`) should be split into 5 edges); and
  - the number of edges to place toward the center of the patch
    (`gl_TessLevelInner[0] = 5` would be a good choice if
    `gl_TessLevelOuter[...] = 5` and a regular tessellation was desired).

Unlike the vertex or fragment shader, the tessellation control shader has access
to attribute information at _all_ of the vertices of a triangle. The main
responsibility of this shader is setting the `gl_TessLevelOuter` and
`gl_TessLevelInner` variables.

### Tessellation Evaluation Shader

The tessellation _evaluation_ shader takes the result of the tessellation that
the tessellation _control_ shader has specified. This shader is called once for
every vertex output during tessellation (including original corners). It has
access to the attribute information of the original corners (e.g., in our code
`in vec3 pos_es_in[]`) and a special variable `gl_TessCoord` containing the co-ordinate of the
current vertex. Using this information, it is possible to interpolate
information stored at the original corners onto the current vertex: for example,
the 3D position. Like the vertex and tessellation control shader, this shader
can change the 3D position of a vertex. This is the _last opportunity_ to do
that, since the fragment shader cannot.

###
