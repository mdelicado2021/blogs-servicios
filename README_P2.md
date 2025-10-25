# DRONE ON A SEARCH AND RESCUE MISSION
The objective of this exercise is to implement the logic that allows a quadcopter to recognise the faces of missing persons and save their locations for a subsequent rescue operation.

## STATE MACHINE
For this exercise, I have implemented a state machine to manage the drone's behaviour.

Ascend: first, the drone ascends to a predetermined height. The height should not be too high because people's vision may be poor, but it also cannot be too low because it may collide with the waves. This ascent is done outside the main loop since the cameras are not needed, assuming that the starting point is already at the relatively constant height at which the route will be flown.

Centre: the first route is to head towards the given coordinates where the rescue area is supposed to be located. These coordinates 40o16'47.23‘ N, 3o49'01.78’ W are approximately equivalent to centre = (33, -36.25). This conversion has been possible thanks to the fact that we have also been given the conversion of the initial position of the drone 40o16'48.2‘ N, 3o49'03.5’ W, which is equivalent to (0,0). To head towards the point, simply use the HAL.set_cmd_pos(x, y, z, az) position control command.

Spirals: this state is based on performing spirals (increasing curvature) to scan the terrain. To do this, simply assign an increasing vx speed using a parameter.

## VIDEO 
https://youtu.be/gH81Ak0hY90
