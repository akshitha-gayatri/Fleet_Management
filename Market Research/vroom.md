Its an open source route optimizing software which takes care of several route optimization problems like:
- TSP (travelling salesman problem)
- CVRP (capacitated VRP)
- VRPTW (VRP with time windows)
- MDHVRPTW (multi-depot heterogeneous vehicle VRPTW)
- PDPTW (pickup-and-delivery problem with TW)

VROOM can also solve any mix of the above problem types.

So usually the control flow is :
1. Data collection
2. Routing engines (SRM)
3. Optimizstion layer (Vroom)

in Vroom website:
- We can choose multiple vehicles and its starting point and ending point in the map and find the best route. We can even choose only start or end
- We can load the map data by manually choosing places or by loading a json file (preffered)

This has good OSRM integration.

It optimizes path using the following methods after calculating the path:
![vroom](https://github.com/user-attachments/assets/cd11ec55-b12a-49d2-9881-cd0f360cc67a)



The github link for Frontend and backend integration is: https://github.com/VROOM-Project/vroom
