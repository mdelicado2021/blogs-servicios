# Localized vacuum cleaner P1
The objective of this exercise is to implement the logic of a navigation algorithm for an autonomous vacuum cleaner by making use of the location of the robot. The robot is equipped with a map and knows it’s current location in it. The main objective will be to cover the largest area of ​​a house using the programmed algorithm.

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

We need an affine transformation that converts world coordinates `(x_w, y_w)` into grid cell coordinates `(x_c, y_c)`.
For the registration to work correctly, a transformation matrix must be applied, consisting of a translation, a rotation, and the subsequent application of a scale.

![image](https://github.com/user-attachments/assets/bf1e1a8e-a9ce-4b4f-9cee-e1409338979c)


The parameters of the transformation matrix `(tx, ty, a, b, c, d)` are calculated from a linear fit using the correspondence between coordinates and cells as data. This transformation can now be applied to any world coordinate to obtain the corresponding grid cell in the simulation.

## Grid contstruction and visualization

To construct the binary grid, as we estimated earlier, each cell will be 40x40 pixels in size for accuracy. First, the PNG image must be converted to greyscale and binarised. To create the grid, I use the function `create_cell_grid_from_map(binary_map, cell_size_px=CELL_SIZE, occ_threshold=0.15)`, where `occ_threshold` is the minimum proportion of occupied pixels that cells must have in order to be marked as such. This is how to enlarge or shrink obstacles. This function, apart from returning the grid, also returns a dictionary of grid-related data such as the size of each cell in pixels, the dimensions of the grid (number of cells: rows, columns), the total size of the binary map, and a copy of the original map size that will be used later.

```python
    meta = {
        "cell_size_px": cell_size_px,
        "grid_shape": grid.shape,
        "map_shape": binary_map.shape,
        "orig_shape": binary_map.shape
    }
    return grid, meta
```

To display the grid, I used the function `create_visual_grid(grid, cell_size=CELL_SIZE)`, which paints the obstacles and the grid in black and the free spaces in yellow. It returns a numpy array that can then be displayed as follows:

```python
vis = create_visual_grid(grid, CELL_SIZE)
WebGUI.showNumpy(vis)
```

![image](https://github.com/user-attachments/assets/bfbeb670-9803-4d94-8293-90d32dad5b0c)

To paint coloured cells during execution, I made copies of `vis` to display the result with `overlay = vis.copy()`.


## Generate the cell path
To generate the cell path, I simply used the function `generate_bsa_path(grid, current_cell, vis=overlay, cell_size=CELL_SIZE, delay=0.01)`, which explores the map from an initial cell, avoiding obstacles, recording the path followed and the points where it had to backtrack, with the option of displaying the entire process visually if desired. It returns a list of cells to be traversed in order, as well as critical points (where it has no unvisited neighbour, orange) and return points (neighbouring cell of the last cell already visited, green). 

This function is based on the Backtracking Spiral Algorithm (BSA), which combines two main ideas: spiral scanning and backtracking. The robot traverses the environment following spiral trajectories to cover simple regions of free space, bypassing obstacles and marking areas already visited as virtual. When points are detected from which new trajectories could start towards unexplored areas, these are stored as return points. Once a spiral sweep is complete, the robot returns to one of these points by travelling over cells that have already been visited and begins a new spiral. This backtracking mechanism ensures that all accessible areas are covered, thus guaranteeing the completeness of the algorithm even in environments with obstacles. To display the route created:

```python
overlay = vis.copy()
path, return_points, critical_points = generate_bsa_path(grid, actual_cell, vis=overlay, cell_size=CELL_SIZE, delay=0.01)
WebGUI.showNumpy(overlay)
```
[This is the result.](https://youtu.be/9nT0cJBBr0M)

## Navigation
Navigation consists of travelling along the path of cells in such a way that you do not get stuck on obstacles. To do this, I have used several functions. 

The function `go_to_point(x_target, y_target, state, tolerance=TOLERANCE, angle_tolerance=ANGLE_TOLERANCE)` uses proportional control for both linear and angular velocity to travel from the centre of the current cell to the centre of the next one. In addition, it is a simple state machine that can be orienting (with angular but not linear velocity), advancing (with linear but not angular velocity) or finished (both velocities at zero). This is done to avoid commanding both types of velocities at the same time and thus avoid inconsistencies in the route.

On the other hand, we have a navigation sub-algorithm to generate the shortest route between the critical point and the return point so that we can continue with the main path: `navigate_to_cell(grid, start_cell, target_cell)`. [Here is the final behaviour.](https://youtu.be/RQ_kgK8-iJE)


## Some problems and possible improvements

- Cells with obstacles and free cells added manually: since the expansion of obstacles is not exact and depends on a value, some obstacles may be too large and others too small. Therefore, a solution that modifies the cells, even if manual, is quite effective. [An example of the vacuum cleaner trying to access an occupied cell](https://youtu.be/WlXHbOWlGfo)

- Cell size: the ideal cell size would be equal to that of the vacuum cleaner and slightly smaller so as not to leave any space unswept, only the margins with obstacles. The problem is that the vacuum cleaner has a radius of around 35 pixels and, as mentioned above, the image has 1012 pixels and the next smallest divisor of 44 is 22 pixels, which is too small. That is why I have opted for a solution of 44x44 pixels per cell, even though it is somewhat large.

- Slowness: for navigation between the centre of each cell to be accurate, the speed must be limited, especially for long distances, because otherwise the pre-established centre tolerance cannot be
exceeded. [An example of the vacuum cleaner overshooting the target due to its speed](https://youtu.be/eISl8lVgkFo)
