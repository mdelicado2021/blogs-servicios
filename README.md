# Service Robotics

In this repository you will find the blogs explaining the practices of the service robotics subject. They show the whole process of code development, as well as explanatory videos and images and implementation alternatives.

## Coordinate registration and transformation matrix

The first step in this exercise is to use the png map provided to obtain a grid map that allows us to navigate using the BSA algorithm.

To establish an initial relationship between the world coordinates and the png pixels, we need to know that this image is 1012x1012. Therefore, we have estimated that the cells are 44x44 pixels, as this would complete the png image, obtaining a 23x23 grid map.

On the other hand, to establish a relationship between the world coordinates and the cells, a transformation matrix was estimated. To do this, I mapped points to obtain a relationship between them:

We have a set of known correspondences between grid cells and world coordinates:

| Grid Cell | World Coordinate (x, y) |
|-----------|------------------------|
| (0,0)     | (5.12, -3.27)          |
| (0,22)    | (-3.98, -3.57)         |
| (22,0)    | (5.13, 5.55)           |
| (22,22)   | (-3.81, 5.86)          |
| (11,12)   | (0,0)                  |
| (12,15)   | (-1, 1.5)              |
| (7,11)    | (0.87, -0.87)          |
| (2,8)     | (2.01, -3.29)          |
| (9,1)     | (5.04, 0.08)           |
| (17,1)    | (5.13, 3.91)           |
| (8,10)    | (1.3, 2.25)            |
| (19,13)   | (-0.12, 4.03)          |
| (17,17)   | (-1.77, 3.77)          |
| (12,21)   | (-3.73, 1.49)          |
| (7,19)    | (-2.77, -0.71)         |

We need an affine transformation that converts world coordinates `(x_w, y_w)` into grid cell coordinates `(x_c, y_c)`. We assume the simplified form:

### Calculating Parameters



This transformation can now be applied to any world coordinate to obtain the corresponding grid cell in the simulation.
