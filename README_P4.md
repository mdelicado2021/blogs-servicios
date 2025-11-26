# Autonomous Warehouse Navigation with OMPL and State Machine
The objective of this exercise is to implement an autonomous navigation system for a holonomic robot within a warehouse-type environment.
The robot must:

- Generate a valid path avoiding shelves.
- Execute this path using reactive control.
- Redirect itself dynamically based on its current position.

To do this, several technologies are combined:

- Map processing (with OpenCV).
- World ↔ map transformations.
- Path planning with OMPL.
- Tracking control.
- State machine that orchestrates the entire process.


## Motion Planner
BIT* (Batch Informed Trees), an optimal sampling-based planner included in OMPL, has been used for planning.

### Why BIT*?
BIT* was chosen over others (RRT, RRT*, PRM, etc.) for several reasons:

- It explores space in a more orderly manner than [*RRTConnect Planner*](https://youtu.be/6i9KIL29UQg), reducing search times.
- It is optimal: it improves the solution over time.
- It performs well in environments with complex obstacles, such as narrow warehouses.
- It generates smoother paths than RRT/RRTConnect.

It is more stable and repeatable on maps with very pronounced aisles and shelves.
In environments such as a warehouse — narrow, with aisles and gaps — BIT* makes good use of the structure and usually finds valid paths in very few cycles.


## Map Processing
Before planning, the map must be converted into a navigable representation:

- Map conversion → greyscale
```python
gray[i, j] = np.average(map_img[i, j]) * 127
```
- Binarisation: light areas (shelves) are marked as obstacles.
```python
binary[gray < 100] = 127
binary[gray >= 100] = 0
```
- Inflation of obstacles: to ensure safety, expansion is applied
```python
inflated = cv2.dilate(binary, kernel)
```
This prevents the planner from passing too close to shelves.

## Coordinate Transformations
The map is in pixels, but the robot operates in world coordinates.
Therefore, a transformation matrix `T` is calculated that allows:
- Converting world points to map points (world_to_map)
- Converting map points to world points (map_to_world)
- The transformation is obtained by a linear adjustment between four known points.

## Key Functions
The most relevant functions of the system are described below:

### build_transform()
Calculates the affine transformation matrix between world and map coordinates.
Allows positions in metres to be converted to positions in pixels.

### delete_pos()
‘Removes’ a specific shelf from the map, deleting it from the inflated mask.
This allows the robot to access a point that was originally blocked.

### is_state_valid(state)
Validity function for OMPL. Checks:
- That the state does not go beyond the map boundaries.
- That the point is in a cell free of inflated obstacles.
- Essential for any sampling-based planner.


### plan_path(start_map, goal_map)
Function that:
- Converts map → world positions.
- Configures an SE2 space.
- Defines warehouse boundaries.
- Creates a `SpaceInformation` with state validator.
- Instantiates the BIT* planner.
- Solves the planning with a timeout of 0.5 s.
- Smooths the path with B-Spline.
- Converts the resulting path back to pixels for display.


### follow_path_step(path_world)
Implements a simple proportional controller:
- If there is a large angular error → corrects yaw.
- If correctly oriented → advances to the next point.
- When a waypoint is considered reached (dist < 0.15), it is removed from the list.
When the list is complete, the robot reaches the target.


### State Machine
The entire navigation process is structured using a state machine that organises the phases of the robot's movement.

### PLAN_PATH_1
This is the initial state.
- The current position is captured.
- A path to the target shelf is calculated using BIT*.
- The path is displayed in WebGUI.
- It is transformed into world points.
When the path is valid → it moves to `FOLLOW_1`.


### FOLLOW_1
It executes point-to-point tracking of the path:
- It calls `follow_path_step()` in each iteration.
- If the current waypoint is reached → it is removed.
- If the list is empty → the robot has arrived.
At that point, it usually transitions to a second plan, a pick-up or another action depending on the practice.


### Why Updating Inside the Loop?
The main process (pose reading, tracking, arrival check) must be inside the loop so that:
- The robot's position is updated in real time.
- The tracker can recalculate speeds at each iteration.
- Paths can be replanned if something changes.
- The state transition is reactive.
Each tick of the loop corresponds to a ‘control iteration’.


### Summary of Behaviour
Overall, the system:
- Processes and expands a warehouse map.
- Configures a transformation between the world and the map.
- Uses BIT* for planning.
- Smooths the path.
- Follows the path using reactive control.
- Communicates with WebGUI to visualise it.
- Orchestrates everything using a state machine.
- Constantly recalculates the position within the main loop.


## Demonstration videos
[*Path followed (shelve 0) with BITstar planner*](https://youtu.be/Vl1kNkpKlRg)

[*Path followed (shelve 1) with BITstar planner*](https://youtu.be/JYGNksctC8Q)

