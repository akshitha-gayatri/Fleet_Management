# Grasshopper Optimization Algorithm

## Overview
The Grasshopper Optimization Algorithm (GOA) is a nature-inspired optimization technique based on the swarming behavior of grasshoppers. It is a metaheuristic algorithm that mimics the social interactions and movement patterns of grasshoppers to solve optimization problems. The algorithm is designed to balance exploration and exploitation of the search space by combining social forces, gravity, and wind effects to guide the search process.

## Algorithm Fundamentals

### Mathematical Foundation
The GOA models grasshopper behavior using:
```python
Xi_d = Si + Gi + Ai
```
Where:
- Xi_d: Position of i-th grasshopper in d-th dimension
- Si: Social interaction (attraction/repulsion)
- Gi: Gravity force (towards target)
- Ai: Wind advection (random perturbation)

### Core Components

Social Interaction Function:
```python
def social_interaction(distances):
    return f*exp(-distances/l) - exp(-distances)
```

Position Update Formula:
```python
def update_position(current_pos, target_pos, c_param):
    """
    Update the position of a grasshopper based on social forces, gravity, and wind.
    """
    new_pos = c_param * (social_forces + gravity_component + wind_component) + target_pos
    return new_pos
```

## Implementation Details

### Basic Structure
```python
class GrasshopperOptimizer:
    def __init__(self, n_grasshoppers, max_iterations, lb, ub):
        """
        Parameters:
        - n_grasshoppers: Number of grasshoppers in population
        - max_iterations: Maximum number of iterations
        - lb: Lower bounds of search space (lower boundary)
        - ub: Upper bounds of search space (upper boundary)
        """
        self.population = self._initialize_population(n_grasshoppers, lb, ub)
        self.max_iterations = max_iterations
        self.c_max = 1.0            # Maximum step size
        self.c_min = 0.00001        # Minimum step size
        self.f = 0.5                # Intensity of attraction
        self.l = 1.5                # Attractive length scale
        
    def optimize(self, objective_function):
        """
        Optimize the objective function using GOA.
        """
        for iteration in range(self.max_iterations):
            c = self._update_c_parameter(iteration)
            self._update_positions(c)
            self._evaluate_fitness(objective_function)
```

## Available Implementations

### Python Implementation
Available on PyPI:
```bash
pip install pyGOA
```

Usage example:
```python
from pygoa import GOA

def objective_function(x):
    return sum(x**2)  # Example objective

optimizer = GOA(
    objective_function,
    lb=-100,
    ub=100,
    dim=30,
    n_grasshoppers=100,
    max_iter=1000
)

best_solution = optimizer.optimize()
```

### Other Implementations
- MATLAB implementation by original authors
- Java implementation in jMetalPy framework
- R implementation in metaheuristicOpt package

## Computational Complexity

- Time complexity: O(n²) per iteration
- Space complexity: O(n*d) where n=grasshoppers, d=dimensions


## Integration Guidelines

### Data Preparation
```python
def prepare_route_data(locations, constraints):
    # Convert to numpy arrays for efficiency
    locations = np.array(locations)
    
    # Normalize coordinates
    normalized_locs = (locations - locations.min()) / (locations.max() - locations.min())
    
    return normalized_locs, constraints
```

### Constraint Handling
```python
def apply_constraints(solution, constraints):
    # Time window constraints
    if not check_time_windows(solution):
        return False
        
    # Capacity constraints
    if not check_capacity(solution):
        return False
        
    return True
```

## Optimization Tips

### Parameter Tuning
Recommended ranges:
- n_grasshoppers: [30, 100]
- max_iterations: [500, 2000]
- c_max: [0.8, 1.0]
- c_min: [0.00001, 0.0001]

### Performance Improvements
- Implement local search
- Use bounded memory for large instances
- Parallelize fitness evaluation
- Cache distance calculations

## Documentation
- Original paper: "Grasshopper Optimisation Algorithm: Theory and Application" (2017)
- Implementation guide: https://github.com/7ossam81/EvoloPy

## Known Limitations

### Technical Constraints
- Sensitive to parameter settings
- May converge prematurely
- Memory intensive for large problems
- Limited theoretical analysis available

### Mitigation Strategies
- Implement restart mechanisms
- Use adaptive parameters
- Employ memory management techniques
- Combine with local search


### Detailed Implementation

```python
from typing import List, Tuple, Callable
import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass

@dataclass
class GrasshopperParams:
    c_max: float = 1.0
    c_min: float = 0.00001
    f: float = 0.5
    l: float = 1.5

class GrasshopperOptimizer:
    def __init__(
        self, 
        n_grasshoppers: int,
        max_iterations: int,
        lb: float,
        ub: float,
        dimensions: int = 30,
        params: GrasshopperParams = GrasshopperParams()
    ):
        """Initialize Grasshopper Optimizer
        
        Args:
            n_grasshoppers: Population size
            max_iterations: Maximum iterations
            lb: Lower boundary of search space
            ub: Upper boundary of search space 
            dimensions: Problem dimensions
            params: Algorithm parameters
        """
        self.validate_inputs(n_grasshoppers, max_iterations, lb, ub)
        self.n_grasshoppers = n_grasshoppers
        self.max_iterations = max_iterations
        self.lb = lb
        self.ub = ub
        self.dimensions = dimensions
        self.params = params
        self.best_solution = None
        self.best_fitness = float('inf')
        self.fitness_history = []
        
        # Initialize population
        self.population = np.random.uniform(
            low=lb,
            high=ub,
            size=(n_grasshoppers, dimensions)
        )

    def validate_inputs(self, n_grasshoppers: int, max_iterations: int, lb: float, ub: float) -> None:
        """Validate input parameters"""
        if n_grasshoppers < 2:
            raise ValueError("Population size must be at least 2")
        if max_iterations < 1:
            raise ValueError("Max iterations must be positive")
        if lb >= ub:
            raise ValueError("Upper bound must be greater than lower bound")

    def social_interaction(self, distances: np.ndarray) -> np.ndarray:
        """Calculate social forces between grasshoppers"""
        return self.params.f * np.exp(-distances/self.params.l) - np.exp(-distances)

    def update_position(self, current_pos: np.ndarray, target_pos: np.ndarray, c_param: float) -> np.ndarray:
        """Update grasshopper position"""
        # Calculate distances between grasshoppers
        distances = np.linalg.norm(self.population - current_pos, axis=1)
        
        # Social forces
        social_forces = np.sum(self.social_interaction(distances))
        
        # Update position
        new_pos = c_param * social_forces + target_pos
        
        # Clip to boundaries
        return np.clip(new_pos, self.lb, self.ub)

    def optimize(self, objective_function: Callable) -> Tuple[np.ndarray, float]:
        """Run optimization
        
        Args:
            objective_function: Function to minimize
            
        Returns:
            Tuple of best solution and fitness
        """
        for iteration in range(self.max_iterations):
            # Update step size
            c = self.params.c_max - iteration * (
                (self.params.c_max - self.params.c_min) / self.max_iterations
            )
            
            # Evaluate population
            fitness = np.array([objective_function(pos) for pos in self.population])
            
            # Update best solution
            min_fitness_idx = np.argmin(fitness)
            if fitness[min_fitness_idx] < self.best_fitness:
                self.best_fitness = fitness[min_fitness_idx]
                self.best_solution = self.population[min_fitness_idx].copy()
            
            # Store history
            self.fitness_history.append(self.best_fitness)
            
            # Update positions
            target = self.best_solution
            for i in range(self.n_grasshoppers):
                self.population[i] = self.update_position(
                    self.population[i],
                    target,
                    c
                )
            
            # Progress logging
            if iteration % 100 == 0:
                print(f"Iteration {iteration}: Best fitness = {self.best_fitness:.6f}")
                
        return self.best_solution, self.best_fitness

    def plot_convergence(self) -> None:
        """Plot convergence history"""
        plt.figure(figsize=(10, 6))
        plt.plot(self.fitness_history)
        plt.title('Convergence History')
        plt.xlabel('Iteration')
        plt.ylabel('Best Fitness')
        plt.yscale('log')
        plt.grid(True)
        plt.show()
```