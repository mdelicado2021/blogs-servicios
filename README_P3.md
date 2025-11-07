# Autonomous Car Parking with Laser Sensors

The objective of this exercise is to implement an autonomous parking behaviour for a vehicle using laser sensors and a finite state machine. The car must identify a free parking space, determine whether there is a vehicle behind, and then execute the appropriate manoeuvre to park accurately.


## State Machine

To manage the car's behaviour, a **state machine** has been implemented, where each state corresponds to a specific phase of the parking process.

### **AUX**
This is the initial state.  
It checks whether there is a car behind using the right laser.  
Depending on this, the manoeuvre will be adapted later:
- If no car is detected behind → it will park moving forward.
- If there is a car → it will perform a reverse parking manoeuvre.


### **ACERCANDO_DCHA** (Approaching on the right)
The car moves **forward** while **turning right**, positioning itself parallel to the row of parked cars.  
When the pose or lateral distance matches the expected configuration:
- If there is no car behind → it transitions to `ACERCANDO_IZQ`.
- If there is a car → it checks the distance to the side and also moves to `ACERCANDO_IZQ` once aligned.


### **ACERCANDO_IZQ** (Approaching on the left)
The car moves **forward** and **turns left** to align its yaw with the line of parked cars.
- When the car reaches the correct orientation (`POSE_RECTO`), it moves to:
  - `MAN_4` if there is no car behind.
  - `BUSCANDO_HUECO` if there is a car behind and the system must search for a free space.


### **BUSCANDO_HUECO** (Searching for a gap)
The car moves straight ahead while scanning the side with the **right laser**.  
The function `detectar_hueco()` determines if there are at least **70 consecutive `inf` values** (no obstacle) in the range `[HUECO_INTERMEDIO_MIN, HUECO_INTERMEDIO_MAX]`.  
When such a gap is found, the state changes to **HUECO_DETECTADO**.


### **HUECO_DETECTADO** (Gap detected)
After detecting the intermediate gap, the car continues forward until it also detects a **rear gap** using a different laser zone.  
Once both have been confirmed, it transitions to **MAN_0**, starting the actual parking manoeuvre.


### **MAN_0 → MAN_4** (Parking manoeuvres)
This sequence of manoeuvres (`MAN_0`, `MAN_1`, `MAN_2`, `MAN_3`, and `MAN_4`) carries out the complete parking process.  
Each state has specific **linear (`V`)** and **angular (`W`) velocities**, as well as **conditions based on pose and distances** to determine when to move to the next one.

- **MAN_0** → Small rotation to start entering the parking space.  
- **MAN_1** → Moves backwards until the back laser detects proximity to the car behind (`MEDIA_MAN_1`).  
- **MAN_2** → Reverse rotation to adjust alignment (`VEL_ANG_MAN_2`).  
- **MAN_3** → Forward correction until the car’s yaw matches `YAW_MAN_3`.  
- **MAN_4** → Final adjustment forward or backward depending on whether there’s a car behind:
  - If there’s a car: stops when x ≤ `POSE_X_MAN_4`.
  - If not: moves until a certain front distance is reached (`MEDIA_MAN_4`).


### **FIN**
The car stops (`HAL.setV(0.0)` and `HAL.setW(0.0)`), concluding the parking manoeuvre.


## Key Functions

### **`detectar_hueco(laser_values, rango_inicio, rango_fin, umbral_consecutivos)`**
Detects whether there are a certain number (`umbral_consecutivos`) of consecutive `inf` values in a given angular range.  
Used to determine the presence of a free space.

### **`no_hay_coche_atras(laser_values)`**
Determines whether there is a car behind by counting the number of `inf` readings in the rear laser.  
If more than 120 infinite values are detected, it assumes there’s no car behind.


## Adjusted Parameters

Several constants control the car’s motion precision and detection thresholds:
- `VEL_LIN`, `VEL_ANG`: base linear and angular velocities.
- `HUECO_INTERMEDIO_MIN`, `HUECO_INTERMEDIO_MAX`: range for intermediate gap detection.
- `RAYOS_CONSECUTIVOS_ZONA_INTERMEDIA`: number of consecutive `inf` rays to confirm a free space.
- `MEDIA_MAN_X`: mean distance thresholds used during each manoeuvre to decide transitions.

These parameters have been tuned experimentally for smoother and more realistic behaviour.


## Some problems

- **Sensor noise**: occasional false readings (finite values inside gaps) can break the detection of consecutive `inf` values, causing unstable detection.
- **Pose drift**: yaw and position readings can slightly desynchronize, affecting the accuracy of transitions between manoeuvres.
- **Parameter sensitivity**: small changes in `MEDIA_MAN_1` or angular velocities can make the manoeuvre too abrupt or too wide.
- **Dependence on environment geometry**: the gap detection assumes a straight row of cars; curves or irregular shapes can mislead the algorithm.


## Demonstration videos
[*Autoparking with car in front and behind*](https://youtu.be/a5r_pbUE-5s)  
[*Autoparking with car behind*](https://youtu.be/MN6fJgEVtt4)  
[*Autoparking with car in front*](https://youtu.be/pZuAdvUak8w)

