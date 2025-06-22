# nyc-taxi-dispatching-optimization
Ride Hailing Optimization Report
Group 6: Kai Yang, Bilwa Khaparde, Linh Tran, Sangita Poudel, Rifa Safeer Shah

INTRODUCTION 

In the ever-evolving landscape of urban mobility, ride-hailing services such as Uber and Lyft have transformed how people travel in metropolitan cities like New York City. Behind the scenes of these services lies a complex dispatching challenge: How do we assign the most suitable cab to each passenger request in real time, factoring in constraints such as traffic, legal zones, driver capacity, and operational costs?
This project explores that challenge using a real-world NYC taxi dataset and formulates a Gurobi-based optimization model that emulates intelligent dispatch decisions under regulatory and operational constraints.
PROBLEM & OBJECTIVES
The rise of ride-hailing platforms like Uber and Lyft has intensified the need for efficient and fair taxi dispatch systems, especially in large metropolitan areas like New York City. The New York City Taxi and Limousine Commission (TLC) collects and provides rich datasets of all yellow and green taxi trips across the five boroughs, including time, location, fare, and cab attributes.

This project focuses on optimizing real-time dispatching of cabs to passenger requests using historical NYC Yellow and Green Taxi trip data. The challenge is not merely to assign the nearest available cab to each passenger, but to do so in a way that:
Maximizes overall profit across all assignments
Respects real-world constraints such as:
Passenger capacity of the cab
Legally permissible pick-up zones based on cab type (yellow vs. green)
Time to arrival (ETA)

Unlike many simplistic dispatch models that assume perfect freedom of movement, this project realistically models the constraints of NYC's regulatory environment, where, for example, green cabs are restricted from street pickups in central Manhattan below 96th Street.

Using Gurobi’s optimization solver, the problem is modeled as a binary assignment problem with constraints, allowing us to explore optimal cab-guest pairings based on multiple factors. The model makes decisions that may defy intuitive assumptions , such as skipping the closest cab in favor of one that better fits all constraints ,thus mimicking the sophisticated decision logic expected in a modern intelligent transportation system.

This problem exemplifies the complexity of urban logistics and mobility, where regulatory, geographic, and economic factors interact, and optimization tools like Gurobi can enable smarter, more transparent dispatch decisions.
DATA
# Data Sources
NYC Open Data – 2016 Yellow & Green Taxi Trip Records
Google Maps Distance Matrix API – Real-time ETA and distance
Simulation of Problem Space
Since there is no explicit data on ride hailing records with exact location of riders and drivers, we simulated a scenario using real-world data.
Demand: Trips requested between 8:00–8:05 PM on Jan 1, 2016
Supply: Trips that ended between 7:55–8:00 PM on Jan 1, 2016
We used a subset of the NYC Yellow and Green Taxi trip records, which includes key features:
Pickup and dropoff latitude/longitude
Trip duration and distance
Fare amounts
Taxi types (Yellow or Green)
Vehicle capacity (seats)
Pickup zones mapped to NYC borough regulations
Additional features were computed:
ETA (Estimated Time of Arrival) using Google Maps API


Distance for route cost approximation (retrieved from Google Maps, thus this is traffic distance)
Parameters: miles per gallon as per type of taxi cab, gas price of 3.65 dollars per gallon for midgrade
Profit = fare -  pickup_cost - riding_cost (estimated from distance of route retrieved from Google Maps API)
Fare = fare amount from demand data 
Pickup cost = fuel cost to drive cab from its current location to the guest's pickup location
Ride cost = fuel cost to drive from the pickup location to the drop-off location)
Data Preparation
Libraries used
Pandas, NumPy: Data cleaning and manipulation
Google Maps: Real-world travel time and route distance estimation
Folium: Interactive mapping and assignment visualization
Gurobipy: Mathematical modeling and optimization using Gurobi
# MODEL FORMULATION
i. Sets:
C: Set of all available cab drivers
G: Set of all available guest (rider) requests 
A⊆C×G:: Valid cab-guest assignment after capacity constraint and zone restrictions

ii.Parameters: 
ETAcg​: Estimated time taken for cab  c∈C to reach guest g∈G.
            Profitcg: Net profit for assigning cab 
           Pg: Number of passengers in guest g’s request
           Capc:Seat capacity of cab c
           MPGc : Miles per gallon of cab c 
           Fuel Price: Assumed cost per gallon of  fuel

Decision Variables
Xcg= 1 (if cab is assigned to guest g
Xcg= 0 otherwise 

iii.Objective Functions
Maximize Total Profit
Max ∑  Profit cg​⋅Xcg​
Minimize Wait Times
Min ∑  ETA cg​⋅Xcg​
iv.Constraints 
Capacity Constraint:Xcg= 0 if Pg > Capc 
It ensures a cab can only serve a guest if the seating capacity is enough to group size
One accepted ride request per cab: ∑​ Xcg ≤1  
                                                         g∈G
It ensures each cab can only be assigned to at most one guest per dispatch cycle — this prevents double booking.
One cab assigned to each ride request: ∑  Xcg=1
                                                             c∈C
It ensures each guest is served by one cab no more no less. 
Green Cab Zone Restriction: Xcg​=0 if cab c is green and guest g is in restricted zone
It ensures green taxis are not allowed to pick up passengers in certain Manhattan south of East 96th Street and west of 110th Street. 
Optimization Logic (Gurobi)
In managing the tradeoff between our dual objectives, we have adopted a strategic approach that aligns with our current business priorities. At this stage of the project, our primary focus is on driving profitability, as establishing a strong financial foundation is critical for long-term sustainability. However, we recognize that customer satisfaction, reflected in shorter wait times, remains an important factor in fostering loyalty and repeat business.

To balance these objectives, we used weights to create a linear combination of the two goals. By assigning a weight of 15 to profit maximization and a weight of 5 to wait time minimization, we ensure that the Gurobi prioritizes profitability while still accounting for operational efficiency. 
RESULTS & INSIGHTS 
Our optimization model was implemented using Gurobi to assign each guest to a feasible cab while maximizing total profit and respecting real-world constraints such as capacity and zoning legality. The final solution clearly demonstrates the effectiveness and practicality of the model in a real-world NYC scenario.
Optimized Assignments
The optimized output matched each guest to exactly one cab, ensuring full coverage of the demand pool. Key examples from the assignment results include:
Guest32 assigned to Cab218
Guest19 assigned to Cab289
Guest254 assigned to Cab240
Guest179 assigned to Cab132
Guest56 assigned to Cab8
Guest30 assigned to Cab260


These assignments were visualized on a map, confirming that the model not only considered proximity but also enforced policy constraints, cab capacities, and profit optimization. The visual representation allowed us to easily validate the logical correctness of the matches and observe spatial distribution.
Challenging the Model’s Optimality
To further validate the model’s intelligence, we examined a particularly interesting assignment: Guest179 → Cab132. On the map, another cab — Cab14 — appeared to be closer to Guest179. This raised a natural question: Was Gurobi’s choice truly optimal?
Deeper Analysis: Guest179 vs. Cab14 vs. Cab132
We conducted a feature-level comparison of the two cabs for Guest179:
Factor
Cab14
Cab132
ETA
469 seconds
778 seconds
Profit
$6.80
$6.52
Capacity
4 seats
5 seats
Guest179 requirement
5 seats
5 seats

While Cab14 had both a shorter ETA and a higher profit, it failed the capacity constraint , it could only hold 4 passengers, while Guest179 required seating for 5. This constraint rendered the assignment infeasible.
Gurobi’s model correctly rejected Cab14 and selected Cab132, which fully satisfied all feasibility rules. This example highlights the strength of the optimization logic: it doesn’t just pick the nearest or most profitable option, but evaluates all constraints holistically to make a valid and optimal decision.
Key Insight
This case study reinforces that our model replicates how intelligent dispatch systems must operate in the real world ,making assignment decisions based not only on profit or proximity but on complete feasibility. Such behavior is critical for building trustworthy systems that perform under real operational rules.
How is Our Model Relevant and solves real world Problems
Real-World Travel Time
We use Google Maps API to get actual driving distances and ETAs , not just straight-line distances , making the assignments realistic and traffic-aware.
Policy Compliance
Our model enforces NYC TLC rules, such as restricting green taxis from picking up below 96th Street in Manhattan, ensuring all assignments are legally valid.
Cost-Aware Decisions
We calculate net profit for each ride by subtracting estimated fuel costs from fare revenue, so assignments are financially optimal , not just fast.
Smart Trade-Offs
The model chooses cabs that best balance speed, capacity, legality, and profit , not just the nearest one , mimicking how real dispatch systems make decisions.
Future Scope & Cross-Industry Potential
While our current model is tailored for NYC taxi dispatching, its core logic ,matching supply to demand under constraints ,has powerful applications in both future mobility systems and other industries. Below are key areas for future development and broader relevance:
Real-Time Dynamic Dispatching: Extend the static model into a live system that updates as new ride requests or cabs become available.
Ride Pooling: Introduce shared rides to increase occupancy and reduce per-rider cost, turning the problem into a more complex routing scenario.
Demand Forecasting: Use historical data to predict high-demand zones and reposition cabs proactively.
Learning-Based Strategies: Use reinforcement learning to improve decision-making over time based on system feedback.
App Integration: Package the model into an API or backend service for real-world ride-hailing platforms.
Live Traffic Map:. This integration could further be done by adjusting the Google Maps API to whether real-time or a specific time period of the day (slow hours, rush hours, accidents,  etc.) to adjust for route ETA, as well as route design.
Cross-Industry Applications
The same optimization framework can be adapted to several industries, including:
Logistics & Delivery (e.g., Amazon, FedEx): Assigning drivers to delivery routes efficiently.
Emergency Response (e.g., ambulances, fire units): Matching emergency vehicles to calls with minimal response time.
Field Services (e.g., utility repairs): Scheduling technicians to service requests by location and skill.
Event Staffing: Assigning staff or volunteers to roles based on availability and proximity.
Learnings
How to translate real-world dispatch logic into LP constraints
Use of multi-objective optimization and Gurobi’s priority system
API integration (Google Maps) for dynamic traffic-aware decision-making








