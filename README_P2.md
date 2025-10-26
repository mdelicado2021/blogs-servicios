# Drone on a search and rescue mission
The objective of this exercise is to implement the logic that allows a quadcopter to recognise the faces of missing persons and save their locations for a subsequent rescue operation.

## State machine
For this exercise, I have implemented a state machine to manage the drone's behaviour.

- **Ascend**: first, the drone ascends to a predetermined height. The height should not be too high because people's vision may be poor, but it also cannot be too low because it may collide with the waves. This ascent is done outside the main loop since the cameras are not needed, assuming that the starting point is already at the relatively constant height at which the route will be flown.

- **Centre**: the first route is to head towards the given coordinates where the rescue area is supposed to be located. These coordinates `40o16'47.23‘ N, 3o49'01.78’ W` are approximately equivalent to `centre = (33, -36.25)`. This conversion has been possible thanks to the fact that we have also been given the conversion of the initial position of the drone `40o16'48.2‘ N, 3o49'03.5’ W`, which is equivalent to `(0,0)`. To head towards the point, simply use the `HAL.set_cmd_pos(x, y, z, az)` position control command.

- **Spirals**: this state is based on performing spirals (increasing curvature) to scan the terrain. To do this, simply assign an increasing vx speed using a parameter `vx_grow_per_s`.

- **Arrived**: When the battery is running low (20%), it enters the arrived state. Here, the only command is for the drone to return to its initial position (the boat).

- **Land**: the drone lands on the boat with `HAL.land()`.

## Some problems

- **Identification radius adjustment**: to prevent the drone from detecting the same person multiple times and thus causing many people to be found, since it would count them as different people when in fact they are the same, I have implemented a radius that generates circles around the first detection so that if more people are detected, they are only considered to be the same person. The problem is that we need to find the value that makes this function possible without it being the case that when there are two people very close to each other, they are still understood to be the same person.

- **Spiral growth adjustment**: in order for the spiral to grow so that the area is completely scanned in a reasonable amount of time, the growth of the spiral must be measured.

- **Height correction**: the height of the drone can be affected by different causes, therefore, if it is detected that this height is sufficiently different from the established height, an instant correction is made to return to the original height.

- **Tolerances**: to confirm that the drone has reached the target location and to adjust heights, there are different reasonable tolerances for the position.

- **Face detection**: the face detector only works if the face is aligned with the image (if it is rotated, it does not detect it). Therefore, in order for faces to be detected satisfactorily without depending on the angle at which they appear in the image, I have rotated the image eight times so that detection is possible in most situations with one of those angles.


## Demonstration video 
https://youtu.be/gH81Ak0hY90


