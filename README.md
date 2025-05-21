# Fleetix: Project Overview

## Problem Statement
Fleetix addresses open source route optimization B2B services for employee transportation, specifically tackling scenarios involving multiple pickups and drop-offs.

## System Overview
This comprehensive solution optimizes employee transportation management with features including:
- Route optimization for one-to-one, many-to-one and many-to-many scenarios
- Geocoding of employee locations
- Road network graph representation
- Constraint handling (vehicle capacity, time windows, route duration)
- Visualization of optimized routes on a map

## Technology Stack
- **Frontend**: Next.js with TypeScript
- **Backend**: Next.js
- **Database**: PostgreSQL
- **Mapping**: OpenStreetMap (OSM)
- **Route Optimization**: Open Source Routing Machine (OSRM)



# Repository Structure

```
.
├── README.md
├── Archive/                  # Previous versions of documentation
├── Code/                     # Source code for the project
│   ├── backend/              # Backend services and APIs
│   │   ├── db/               # Database connection and utilities
│   │   ├── entities/         # Database entity definitions
│   │   ├── routes/           # API endpoint definitions
│   │   ├── services/         # Business logic implementations
│   │   └── src/              # Other source files
│   ├── frontend/             # Next.js frontend application
│       ├── public/           # Static assets
│       └── src/              # Frontend source code
│           ├── app/          # Next.js app directory
│           ├── components/   # Reusable UI components
│           ├── pages/        # Page components
│           └── utils/        # Utility functions
│   
├── Docs/                     # Project documentation
│   └── Status_trackers/      # Weekly status tracking documents
├── Market Research/          # Research on tools and technologies
.
```

# Documentation and Resources

All necessary documentation and resources for the project can be found in the following directories:

- **[`Important Documents`](./Docs/)**: Contains design documents, project plans, and presentations.
  - [SRS Document](./Docs/SRS_Document.pdf): System Requirements Specification.
  - [Project Plan](./Docs/Project_Plan.pdf): Project planning and timeline.
  - [Design Document](./Docs/Design_Document.pdf): Detailed design specifications.
  - [Status Trackers](./Docs/Status_trackers/): Weekly status tracking documents. Access the final status tracker [here](./Docs/Status_Tracker.pdf).
  - [Test Tracker](./Docs/Test_Tracker.pdf): Document for tracking test cases and results.
  - [R1 Presentation](./Docs/R1_Presentation.pdf): Presentation for the first release.
  - [Final Presentation](./Docs/Final_Presentation.pdf): Final presentation of the project.

- **[`Market Research`](./Market%20Research/)**: Includes research on tools and technologies used in the project.
  - [Cost Analysis](./Market%20Research/CostAnalysis.md): Detailed pricing overview of Google Maps Platform services.
  - [Data Acquisition](./Market%20Research/DataAcquisition.md): Documentation on acquiring and working with OSM data.
  - [Geocode](./Market%20Research/Geocode.md): Information about geocoding employee addresses with code examples.
  - [Google API](./Market%20Research/GoogleAPI.md): Analysis of Google Maps Platform costs and features.
  - [Grasshopper](./Market%20Research/Grasshopper.md): Details about the Grasshopper Optimization Algorithm.
  - [Heuristics](./Market%20Research/Heuristics.md): Documentation on heuristic algorithms for route optimization.
  - [VROOM](./Market%20Research/vroom.md): Overview of VROOM for vehicle route optimization.

- **[`Code`](./Code/)**: Source code for the project.
  - [frontend/](./Code/frontend/): Frontend application built with Next.js.
  - [backend/](./Code/backend/): Backend services and APIs.

- **[`Archive`](./Archive/)**: Contains all previous versions of live documents present in the `Docs/` directory.

For any additional information or queries, refer to the `README.md` files in respective directories.
