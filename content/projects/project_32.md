+++
title = "Polygons: SDF"
description = "Sculpting with code/math"
weight = 33
date = "2024-11-11"

[extra]
local_image = "images/sdrect.png"
[taxonomies]
tags=["art","sdf","mathematics","light"]
+++ 

The foundation of this artwork lies in the Signed Distance Function (SDF) technique, where we calculate the minimum distance from any point to a regular polygon's edges. We achieve this by constructing each polygon from a set of vertices placed at equal angular intervals (2π/n, where n is the vertex count), and then computing the perpendicular distance to each edge. The resulting distance field creates a smooth boundary that defines our shape, with positive values outside and negative values inside the shape.
The lighting system is built upon this distance field, using a combination of vertex-based point lights and environmental illumination. Each vertex acts as a light source, with intensity falling off based on distance, while rim lighting is calculated using the dot product between the view direction and surface normal. We enhance depth perception through ambient occlusion, where deeper layers receive less ambient light, and add iridescence by modulating light based on the viewing angle and surface distance.

Download the app by searching "sdrect":

https://github.com/altunenes/rusty_art/blob/master/shaders/sdrect.wgsl