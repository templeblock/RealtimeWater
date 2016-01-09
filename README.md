# Watersimulation

In this project a water-simulation was implemented based on the approach suggested in the presentation "Fast Water Simulation for Games Using Height Fields" by Matthias Müller-Fischer. Decent effort was invested to make the result visually pleasing.

Follow this link to view the presentation: http://matthias-mueller-fischer.ch/talks/GDC2008.pdf
I included the pdf in the repository as well in case of the document being offline in the future.

Everything is rendered with OpenGL 4.3 at real-time frame rates applying a simple ad-hoc shading. 
Multiple rendering passes are used to determine the depth of water and calculating fake refractions of the ground. 
Light is attenuated by the water based on the depth from the point of view of the light source. 
To give the impression of subsurface-scattering and shift in color a look-up texture is used.
Caustics are achieved by projecting a texture onto the ground. Nothing real here either.
To give the impression of small ripples on the water a normals are read from a normal-map.

The water-simulation is implemented in the compute shader.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/1.jpg)

To make the simulation interesting the water is pulled up every few seconds by two gaussian bells.
The normals, which are visualized with the aid a geometry shader, are recalculated after each simulation step. See the following figure.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/2.jpg)

The wireframe-view gives an impression of how dense the grid is. The following figure shows a view from below the surface.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/3.jpg)

When the ground is rendered without the water, you can see how the depth of the water affects the lighting through attenuation. On the right bottom you can be that individual waves are included in the depth-calculation. High waves result in a stronger attenuation and therefore a darker ground. Additionally you can see some distortion in the caustics happen.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/4.jpg)

The shore line did not get much attention. No foam was implemented but could be added based on the depth of the water seen from above. Shallower water would have stronger foam.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/5.jpg)

The following figure shows the water-surface seen from the light-source. To be able to distort the caustics, the normals of the water-surface are used. These can be obtained from this texture.

![alt tag](https://github.com/thehenkk/Watersimulation/blob/master/images/6.jpg)
