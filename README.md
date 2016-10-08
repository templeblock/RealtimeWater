# Watersimulation

In this project a water-simulation was implemented based on the approach suggested in the presentation "*[Fast Water Simulation for Games Using Height Fields](http://matthias-mueller-fischer.ch/talks/GDC2008.pdf)*" by Matthias Müller-Fischer. Some effort was invested to make the result visually pleasing.

I included the [document](https://github.com/hpatjens/Watersimulation/tree/master/documentation/Fast Water Simulation for Games Using Height Fields - Matthias Müller-Fischer.pdf) in this repository as well in case of the document being offline in the future.

This is my first attempt to simulate and render water. This is not a tutorial or even an attempt to do things right. Just keep this in mind when you browse the code.

The water simulation is based on the method that Fischer proposes for 2D heightfields and implemented using the compute shader.
Everything is rendered with OpenGL 4.3 at real-time frame rates applying a simple ad-hoc shading - this is not a PBR renderer.
Multiple rendering passes are used to determine the depth of the water and calculating fake refractions of the ground. 
Light is attenuated by the water based on the distance the light has to travel inside the medium.
To give the impression of subsurface-scattering and shift in color a look-up texture is used.
Caustics are achieved by projecting a texture onto the ground. Nothing real here either.
To give the impression of small ripples on the water a normals are read from a normal-map.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/1.jpg)

## Simulation

The simulation directly operates on the vertices of the water mesh. This ties the simulation and rendering accuracy directly together which can be a drawback when it comes to dynamically scaling the rendering workload to ensure constant framerates. (Which is not happening in this application.) A huge benefit is the possibility to let the vertex and compute shader operate on the same buffer object. While rendering the buffer is interpreted via the *GL_ARRAY_BUFFER* target and for the simulation it is provided to the compute shader via the *GL_STORAGE_BUFFER* target.

These lines implement the above mentioned simulation method:

```cpp
vec4 position = positionsPrev[index];

// f = c^2*(u[i+1,j]+u[i-1,j]+u[i,j+1]+u[i,j-1] – 4u[i,j])/h^2
float f = c * c * (
	positionsPrev[clampedIndex(gl_GlobalInvocationID.xy + uvec2(-1,  0))].y + 
	positionsPrev[clampedIndex(gl_GlobalInvocationID.xy + uvec2( 1,  0))].y + 
	positionsPrev[clampedIndex(gl_GlobalInvocationID.xy + uvec2( 0, -1))].y + 
	positionsPrev[clampedIndex(gl_GlobalInvocationID.xy + uvec2( 0,  1))].y - 
	4.0 * position.y) / (h * h);	

// v[i,j] = v[i,j] + f*∆t
position.w = position.w + f * DeltaTime;

// unew[i,j] = u[i,j] + v[i]*∆t
position.y = position.y + position.w * DeltaTime;
```

The vertices of the water surface are altered by the compute shader using a double buffering. This ensures results that are independent of the execution order of the compute shader threads. *positionsOld* referes to the vertex positions from the previous frame. *position* holds the position of the vertex, that is currently computed. The w component of the position stores the velocity of that water column. The last line alters the height of the current vertex, which is the y-coordinate.

To make the simulation interesting the water is pulled up every few seconds by two gaussian bells.

## Rendering

There are 3 framebuffers on which the rendering operates.

### 1. Watermap
The first framebuffer contains the water surface which is rendered from the position of the light source using an orthographic projection. A depth buffer and a color buffer is rendered. The color buffer is made up of 3 channels that store the normals of the water surface. This buffer has a resolution of 1024x1024.

|Orthographic projection|Normal buffer (depth buffer is not shown)|
|-------|-------|
|<img src="https://github.com/thehenkk/Watersimulation/blob/master/documentation/images/watermap.png" alt="alt text" width="500">|<img src="https://github.com/thehenkk/Watersimulation/blob/master/images/6.jpg" alt="alt text" width="500"> |

### 2. Background 
The ground (without the water on top) is rendered from the viewers position. This framebuffer stores the color of the pixels and the depth from the viewer. This buffer as well as the third is rendered on full resolution.

|Perspective projection|Color buffer (depth buffer is not shown)|
|-------|-------|
|<img src="https://github.com/thehenkk/Watersimulation/blob/master/documentation/images/background.png" alt="alt text" width="500">|<img src="https://github.com/thehenkk/Watersimulation/blob/master/images/backgroundbuffer.jpg" alt="alt text" width="500"> |

### 3. Water 
The water surface is rendered from the viewers position.

|Perspective projection|Color buffer (depth buffer is not shown)|
|-------|-------|
|<img src="https://github.com/thehenkk/Watersimulation/blob/master/documentation/images/water.png" alt="alt text" width="500">|<img src="https://github.com/thehenkk/Watersimulation/blob/master/images/waterbuffer.jpg" alt="alt text" width="500"> |

### Default framebuffer
Finally the second and third buffer are combined and rendered onto the default framebuffer.

### Attenuation
The light gets attenuated by the water on the way to the ground and on the way to the camera. From the watermap we can get the distance the light travels on the way to the ground by projecting the currently processed fragment onto the watermap and calculating the difference to the stored value (w). The attentuation on the way to the viewer can be obtained by calulating the difference between the water fragment and the depth value that is stored in the background buffer (b).

When the ground is rendered without the water, you can see how the depth of the water affects the lighting. On the right bottom you can see that individual waves are included in the depth-calculation. High waves result in a stronger attenuation and therefore a darker ground. Additionally you can see some distortion in the caustics happen.

This is of course a very approximated model that assumes homogenous density distribution and no scattering at all.

|Perspective projection|Color buffer (depth buffer is not shown)|
|-------|-------|
|<img src="https://github.com/thehenkk/Watersimulation/blob/master/documentation/images/attenuation.png" alt="alt text" width="500">|<img src="https://github.com/thehenkk/Watersimulation/blob/master/images/4.jpg" alt="alt text" width="500"> |

Because the simulation constantly changes the water surface, the normals have to be recalculated every frame. The geometry shader is used to verify the calculations.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/2.jpg)

The shore line did not get much attention. No foam was implemented but could be added based on the depth of the water seen from above or the velocities of the water columns in the w component of the vertices.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/5.jpg)
