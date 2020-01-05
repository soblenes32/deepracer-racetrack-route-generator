![A New York Racetrack sample image depicting a route and heading-over-time charts](./assets/new-york-sample.png)
# DeepRacer Racetrack Route Generator
> A genetic algorithm-based simulation approach for generating optimized racetrack traversal routes

The Route Generator takes DeepRacer racetrack data as input and employs a stochastic process to approximate an optimal traversal route, given a few assumptions about the characteristics of the DeepRacer vehicle. The output is an ordered waypoint list registered to the same coordinate system as the input, which may be exported to a file or another format that may be embedded in the DeepRacer reward function. The program itself requires non-trivial computation time, and is not suitable for runtime execution from the reward function.

## Installation

This program can be cloned and directly executed from a default anaconda python3 jupyterlab/notebook install.

## Usage

    //Import package
    //Run commands
    //Sample output

## Motivation & background

The solution space with the DeepRacer in order to arrive at a policy optimized for efficient traversal of the racetrack can be a time and computationally expensive endeavor. By providing the DeepRacer with "hints" about how to navigate the racetrack efficiently in the reward function, the compute required to arrive at a good policy may be reduced. 

## How it works

Prospective vehicle routes are represented as an interpolation over a series of stochastically generated command points. A command point consists of two values: an index, corresponding to the index of an inner track boundary waypoint, and a quantity between \[0, track_width\]. The quantity indicates the distance of the command point from the inner track waypoint, perpendicular to the track edge, as measured between the waypoint and its indexed predecessor. The distance quantity is initialized as a bounded gaussian in order to bias command point values closer to the center of the track.

A cubic beizer curve is employed to interpolate the sequence of command points. Waypoints are generated at regular intervals along the interpolation.

The proposed route may run out-of-bounds. Therefore, a recursive checking routine is executed to resolve out-of-bounds route scenarios. The out-of-bounds check is conducted via a spatial index search (KD tree) that determines the distance between each route waypoint and its closest inner and outer boundary waypoint. If the distance from the route waypoint to either the closest inner or outer track boundary waypoint is greater than the width of the track, then the route waypoint is deemed to lie out-of-bounds. A new command point is generated at the nearest inner track boundary waypoint of the first sequential route waypoint that is deemed out-of-bounds. The interpolation and route waypoints are then recalculated incorporating the new command point, and re-assessed for violation of the track boundaries.

The boundary checking routine is relatively computationally expensive, making a brute-force exploration of the solution space prohibitive to developers of small means. A genetic algorithm is implemented to help offset the cost of route exploration. 

The GA fitness function is calculated based on the distance and relative angle between each pair of route waypoints. It is generally designed to mimic traversal by a vehicle constrained by maximum speed at rate of turn. While a higher-fidelity simulation is certainly possible, insufficient data about the acceleration and cornering speed constraints on either the DeepRacer simulated vehicle or its physical counterpart make this impractical as of time of writing.

The GA mutation function selects one of three operations to perform on the route: remove a command point, add a command point, or adjust the distance on an existing command point

The GA crossover function picks two random cut points corresponding to track inner boundary waypoints, and swaps all command points between each of the two routes.

## Limitations of this approach

The current implementation requires:
- Uniform track width
- Track is a loop
- Inner track waypoint sequences are ordered

Because the search space is large and a genetic algorithm has been employed to help approximate an optimal route, this program probably cannot be effectively used on-line. Rather, an optimal route should be precomputed and loaded for use as part of the DeepRacer activation function.

Data about the DeepRacer vehicle is not currently readily available. The current implementation of the fitness function is a very rough approximation, designed to impose plausible costs of sharp cornering and route inefficiency. The provided implementation assumes instant acceleration and does not simulate tire traction or similar physics concerns. The actual DeepRacer simulation and its physical proxy may experience very different behavior, and therefore may have a substantially different optimal route 

## Acknowledgements
Special thanks to Jorge Silva whose fantastic blog post (https://medium.com/myplanet-musings/the-best-path-a-deepracer-can-learn-2a468a3f6d64) has partially inspired this approach.


### Functional tracks:
- Bowtie_track
- London_Loop_Train
- New_York_Track
- Oval_Track
- Tokyo_Training_Track
- AWS_track
- Virtual_May19_Train_track

### Nonfunctional tracks:
- Reinvent_base => There seems to be a problem with the inner track data waypoint sequence reversing along the y apex
- New_York_Eval_Track => There seems to be a problem with the inner track data waypoint sequence
- H_Track => Algorithm assumes a uniform track width
- Straight_Track => Algorithm assumes circular track. This could be changed if there are nontrivial instances in the future.
