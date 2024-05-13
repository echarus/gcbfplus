# GCBF+

This research-focused GCBF+ branch demonstrates the capability of using external drone state data collected from BlueSky air traffic simulator for collision avoidance evaluation of GCBF+ controlled drones. This GCBF+ branch is for evaluation only and no model training is involved. The name of the Python file that runs the evaluation is "test_bluesky_integration.py". This demo demonsrates a single specific scenario that involves 11 drones. 10 of the 11 drones consist of BlueSky simulated drone data and the first drone is the GCBF+ controlled drone which gets its control actions from the default trained model (under DoubleIntegrator) during the GCBF+ evaluation. No specific arguments are needed to run this test code with the provided external sample BlueSky simulation data since all needed input arguments are hardcoded in the test script ("test_bluesky_integration.py") for simplicity. To run the test code, simply use "python test_bluesky_integration.py" and make sure that you have the required development environment suggested by the main GCBF+ branch.

## About the BlueSky simulations:
- The simulation time step had been set to ‘0.03 seconds’ both in the GCBF+ and in BlueSky simulations for consistency between the two platforms.
- A total of 11 drones are used in the GCBF+ simulation and 10 of the 11 drones correspond to the BlueSky simulated drone data (state data) and the drone type used in the BlueSky simulation is “Mavic”. 
- In the BlueSky scenario, two parallel airspace corridors (from left to right) are simulated where 5 Mavic drones follow each other along each of these two separate air corridors.
- The airspace in the BlueSky simulation data generation is close to KDAL and KDFW airports (Dallas- Texas area).
- In BlueSky simulation, each of the 10 Mavic drones first ascends to an altitude of 10 feet and then moves towards the destination with an airspeed of 5 knots along the corridor path (5 knots is the minimum speed allowed in the BlueSky simulation for Mavic drones).

## About offline preprocessing of BlueSky simulation data:
- The BlueSky lat/long coordinates are converted to UTM x-y coordinate values to be compatible with GCBF+ state data requirements.
- For the single GCBF+ controlled drone ‘Double Integrator’ model is used in GCBF+.
- Because the Double Integrator model in GCBF+ requires velocity (in x and y coordinates) as state information, the velocities (in UTM x and y direction) are computed in an offline-manner from the converted UTM (x,y) drone  state data.
- The BlueSky simulated drone speeds are downscaled to 1 knot (originally 5 knots) to be compatible with the maximum drone speed in GCBF+’s Double Integrator model.
- The BlueSky drone state data information (x, y, vx, vy) at each simulation timestep are stored and saved in a pickle file (can be found 'external_data/bluesky_other_drones_reduced_speed_5.pkl') together with another generated pickle file which contains the start and destination points of the main drone that GCBF+ will be controlling (can be found under 'external_data/main_drone.pkl'). 
- The start and destination points of the GCBF+ controlled main drone are selected such that the main drone crosses paths with the BlueSky drones in the upper air corridor.
- The area-size input argument is set to 55 since the all 11 drones activities are within an area size of 55.

## Integration of BlueSky data and main drone data positions to GCBF+
- The number of agents in GCBF+ simulation is set to 11. Among these 11 agents (or drones), the first agent is considered as the main agent that GCBF+ will be controlling and the rest correspond to the BlueSky simulated drones. For this reason, num-agents argument is set to 11 in "test_bluesky_integration.py". 
> parser.add_argument("-n", "--num-agents", type=int, default=11)
- The pickle file that contains the BlueSky drone state data (can be found under 'external_data/bluesky_other_drones_reduced_speed_5.pkl') and the pickle file that contains the main drone's start and destination positions (can be found under 'external_data/main_drone.pkl') are then read within the GCBF+ platform. The reading process takes place inside the initialization portion of the code, "gcbfplus/env/double_integrator.py"
- In each simulation timestep iteration in GCBF+, the first agent’s state/control information is kept as it is when forming the required graph data for GCBF+ whereas for the remaining 10 agents (drones), the offline BlueSky simulation data (for the corresponding time step) are used. 
- No obstacles are considered in the GCBF+ environment (which is the default case) in GCBF+ simulation.
- Inside GCBF+, the option for "--nojit-rollout" is set to True in order to access the BlueSky simulation data at each timestep during simulation.
- For ffmpeg installation issues in the development environment where this branch is developed, the movie file type is changed to gif inside "test_bluesky_integration.py", however this can be changed back to mp4 format if ffmpeg is available in the environment where this code is going to used.
- The rest of the modifications mostly take place inside "gcbfplus/env/double_integrator.py"
- The used pre-trained model is the default trained model provided with the original GCBF+ code and is under the directory: "pretrained/DoubleIntegrator/gcbf+/models/"

## Other notes 
- The demo code corresponds to a specific scenario that consists of 11 drones where only the first one is being controlled by GCBF+ and the rest of the drones' state data (10 drones) are read from offline simulation data files provided in this branch under the '/external_data/' folder. If one is interested to generalize this code for some other scenarios in which the number of main drones and the number of BlueSky drones are needed to change, one needs to revisit the modified code sections inside "gcbfplus/env/double_integrator.py" such that the correct number of drones are used and their state data are inserted accordingly in the code. For other collision avoidance scenarios, one needs to also create the correct scenario file in BlueSky platform and conduct the simulations in BlueSky followed by UTM coordinate conversion and speed state additions as were done when generating these external BlueSky simulation data. For any questions about this research-based demo code, new scenario generation or BlueSky simulation data generation process, please check with the MITRE team.



