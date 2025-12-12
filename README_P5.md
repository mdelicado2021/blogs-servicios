# Laser Mapping
This Python script is designed to program an autonomous mobile robot, equipped with a laser scanner and odometry (self-localization), to autonomously explore and construct a **Probabilistic Occupancy Grid Map** of its environment (e.g., an industrial warehouse).

The solution integrates advanced mapping and navigation techniques, specifically focusing on **Frontier-Based Exploration** for path planning and a **Log-Odds** model for robust grid map construction.

## Project Objectives

The primary goals of this implementation are:

1.  **Autonomous Exploration:** Program a robust exploration algorithm (Part A) that guides the robot to systematically visit unexplored areas until the entire environment is covered.
2.  **Probabilistic Grid Mapping:** Implement a probabilistic mapping algorithm (Part B) to build the Occupancy Grid Map. This involves:
    * Implementing the **Probabilistic Sensor Model** for the laser.
    * Combining evidence using the **Bayes Rule** in a log-odds framework.
    * Incorporating **saturation mechanisms** to prevent excessive probabilistic inertia.

## 1. Probabilistic Occupancy Grid Mapping (Log-Odds)

The map is represented as a high-resolution 2D array (`log_odds` matrix) where each cell stores the log-odds of being occupied.

### 1.1. Log-Odds Framework

The **Log-Odds ratio** ($L(m)$) is used to represent the state of a map cell $m$ and allows for efficient, additive updating of evidence based on sensor readings.

$$L(m) = \log\left(\frac{P(m)}{1 - P(m)}\right)$$

Where $P(m)$ is the probability of the cell being occupied.

| Parameter | Description | Log-Odds Value |
| :--- | :--- | :--- |
| **Initial State** | Unknown probability $P(m) = 0.5$ | $L_0 = 0.0$ |
| **Occupied Update** | Evidence of an obstacle (e.g., $P_{occ} = 0.9$) | $L_{occ} = \log(0.9/0.1)$ |
| **Free Update** | Evidence of free space (e.g., $P_{free} = 0.3$) | $L_{free} = \log(0.3/0.7)$ |

### 1.2. Sensor Model and Combination of Evidence

The `update_map_with_scan` function processes each laser beam:

1.  **Ray Tracing (Bresenham):** The `bresenham` algorithm is used to determine the set of cells lying between the robot's position and the laser's endpoint (hit point).
2.  **Free Space Update:** All cells along the ray's path (excluding the final hit cell) are updated using the **free evidence** ($L_{free}$).
3.  **Occupied Space Update:** The final cell (hit point) is updated using the **occupied evidence** ($L_{occ}$).
4.  **Axial Approximation:** Given that the laser has a small angular opening, the implementation uses the geometric axial approximation model (ray casting) as a good estimate.

### 1.3. Saturation Mechanism

To limit the influence of noisy or repeated measurements and prevent **probabilistic inertia**, the log-odds values are strictly constrained:

* **Max/Min Limits:** The log-odds values are clamped between pre-defined bounds: $L_{min}$ (highly free) and $L_{max}$ (highly occupied). This ensures that no single cell becomes irreversibly certain.

    ```python
    if log_odds[ly, lx] < L_min: log_odds[ly, lx] = L_min
    if log_odds[ly, lx] > L_max: log_odds[ly, lx] = L_max
    ```
    

## 2. Frontier-Based Exploration Algorithm

To guarantee complete coverage, the robot utilizes a **Frontier-Based Exploration** strategy, which is vastly superior to simple random movement.

### 2.1. The Superiority of Frontier Exploration

A **random navigation** algorithm, where the robot moves and turns arbitrarily, is **incapable of completing the map in a coherent time** for any moderately sized environment.

* **Coherence:** Random walks lack a coherent strategy, causing the robot to revisit already explored areas repeatedly.
* **Completeness:** The probability of a random path passing through the single optimal point required to view the last unexplored corner approaches zero. In complex or cluttered environments, the exploration becomes exponentially inefficient.
* **Frontier-Based** exploration, conversely, is an **information gain maximization** strategy. By targeting the boundary between known and unknown space (the **frontier**), the robot guarantees that every movement yields new information, ensuring systematic and efficient coverage.

### 2.2. Frontier Detection and Selection

1.  **Frontier Identification:** The script identifies frontier cells—cells marked as **free** that are adjacent to at least one **unknown** cell.
2.  **Clustering:** Adjacent frontier cells are grouped into `clusters`.
3.  **Reachability Check:** A Breadth-First Search (`compute_reachable_mask_from_robot`) is performed to ensure the selected frontier is actually accessible from the robot's current position (i.e., not blocked by a wall already mapped).
4.  **Goal Selection:** The center of the largest, closest, and reachable frontier cluster is chosen as the next navigation target.

## 3. Control and Robustness (FSM)

A Finite State Machine (FSM) manages the robot's behavior, ensuring safe and efficient operation:

* **`IDLE`:** Searching for and selecting the next best frontier goal.
* **`NAVIGATING`:** Executing the control law (`compute_control_towards`) to reach the target. Includes a **Stuck Detection** mechanism based on pixel movement to trigger replanning if the robot is trapped.
* **`ESCAPE`:** A recovery state triggered by close obstacles (`obstacle_ahead`). The robot rotates away from the obstacle using an information gain heuristic (`choose_escape_direction`) and moves forward briefly to clear the area, preventing local minima (bouncing off walls).

## 4. Testing Localization Sensitivity

A crucial part of this exercise is testing the map-building algorithm against different sources of robot localization, illustrating the fundamental problem of **SLAM (Simultaneous Localization and Mapping)**.

The `get_pose()` function is parameterized (`USE_ODOM = 1`) to use a localization source.

| Localization Source | `USE_ODOM` Value | Localization Quality | Expected Map Quality |
| :--- | :--- | :--- | :--- |
| **Good Odometry** | `HAL.getOdom()` (Default) | High | Clean, accurate map. |
| **Noisy Odometry** | `HAL.getOdom2()` | Medium Noise | Noticeable blurring and ghosting of walls. |
| **Very Noisy Odometry** | `HAL.getOdom3()` | High Noise | Significant map deterioration, accumulation of large localization errors (drift). |

By switching the `USE_ODOM` parameter to 2 or 3, one can observe directly **how sensitive the map quality is to the precision of the available localization**. When localization quality degrades, the map will show **double images** or **smearing** because the sensor readings are incorrectly mapped to the grid due to erroneous robot position estimates.

## Final Map
![image](https://github.com/user-attachments/assets/7e886483-2340-4e1c-b923-92e49075065a)

## Demonstration videos
- [*Random navigation*](https://youtu.be/W7sDsiG8aEM)
- [*Expected behaviour*](https://youtu.be/s3WbTMg03SA)
