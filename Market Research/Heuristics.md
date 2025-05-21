# Heuristic Algorithms in Route Optimization

## Overview
Heuristic algorithms are approximate solution techniques designed to solve complex optimization problems when exact methods become computationally infeasible. These algorithms are particularly effective for NP-hard problems like the Vehicle Routing Problem (VRP) and its variants, providing practical solutions while balancing computational efficiency and solution quality.

## Algorithm Fundamentals

### Mathematical Foundation
The algorithms typically optimize an objective function subject to constraints:
```python
minimize f(x)
subject to:
    g(x) <= 0  # Inequality constraints
    h(x) = 0   # Equality constraints
    x ∈ X      # Solution space
```

### Core Components

Solution Representation:
```python
def encode_solution(route):
    """Convert route to internal representation"""
    return [city_indices for city_indices in route]
```

Fitness Evaluation:
```python
def evaluate_fitness(solution):
    """Calculate solution quality"""
    return sum(distances[i][j] for i, j in zip(solution[:-1], solution[1:]))
```

## Implementation Details

### Basic Structure
```python
class RouteOptimizer:
    def __init__(self, problem_size, max_iterations, algorithm_type):
        """
        Parameters:
        - problem_size: Number of locations to route
        - max_iterations: Maximum number of iterations
        - algorithm_type: 'GA', 'ACO', or 'SA'
        """
        self.problem_size = problem_size
        self.max_iterations = max_iterations
        self.algorithm = self._initialize_algorithm(algorithm_type)
        self.best_solution = None
        self.best_fitness = float('inf')
        
    def optimize(self, distance_matrix):
        """
        Optimize the routing problem.
        """
        for iteration in range(self.max_iterations):
            candidate = self.algorithm.generate_candidate()
            fitness = self.evaluate_fitness(candidate)
            self.update_best_solution(candidate, fitness)
```

## Available Implementations

### Python Implementation
Available via pip:
```bash
pip install route-opt
```

Usage example:
```python
from route_opt import RouteOptimizer

optimizer = RouteOptimizer(
    problem_size=100,
    max_iterations=1000,
    algorithm_type='GA'
)

best_route = optimizer.optimize(distance_matrix)
```

### Other Implementations
- Google OR-Tools (C++, Python, Java)
- OptaPlanner (Java)
- DEAP (Python)
- jMetal (Java)

## Computational Complexity

- GA: O(population_size * generations * evaluation_cost)
- ACO: O(n²m) where n=cities, m=ants
- SA: O(iterations * neighbor_generation_cost)

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
- GA population size: [50, 200]
- ACO ants: [10, 100]
- SA initial temperature: [100, 1000]
- Iteration limit: [1000, 5000]

### Performance Improvements
- Implement local search
- Use candidate lists
- Cache distance calculations
- Parallelize fitness evaluation

## Documentation
- "A Comprehensive Survey of Recent Advances in Metaheuristics" (2020)
- Implementation guides: https://github.com/google/or-tools

## Known Limitations

### Technical Constraints
- No guarantee of optimal solution
- Performance dependent on parameter tuning
- May get stuck in local optima
- Scaling issues with large problem sizes

### Mitigation Strategies
- Multi-start approaches
- Hybrid algorithms
- Adaptive parameter control
- Parallel implementation

# Heuristic Algorithms in Route Optimization

[Previous sections remain the same...]

### Detailed Implementation

```python
from typing import List, Tuple, Callable, Optional
import numpy as np
from dataclasses import dataclass
from enum import Enum
import random
import math

class AlgorithmType(Enum):
    GA = "genetic"
    ACO = "ant_colony"
    SA = "simulated_annealing"

@dataclass
class OptimizationParams:
    # Genetic Algorithm Parameters
    population_size: int = 100
    crossover_rate: float = 0.8
    mutation_rate: float = 0.1
    
    # Ant Colony Parameters
    n_ants: int = 20
    pheromone_decay: float = 0.1
    alpha: float = 1.0  # Pheromone importance
    beta: float = 2.0   # Heuristic importance
    
    # Simulated Annealing Parameters
    initial_temp: float = 1000.0
    cooling_rate: float = 0.95
    min_temp: float = 1.0

class RouteOptimizer:
    def __init__(
        self,
        problem_size: int,
        max_iterations: int,
        algorithm_type: AlgorithmType,
        params: OptimizationParams = OptimizationParams()
    ):
        """Initialize Route Optimizer
        
        Args:
            problem_size: Number of locations to route
            max_iterations: Maximum iterations
            algorithm_type: Type of algorithm to use
            params: Algorithm parameters
        """
        self.validate_inputs(problem_size, max_iterations)
        self.problem_size = problem_size
        self.max_iterations = max_iterations
        self.algorithm_type = algorithm_type
        self.params = params
        self.best_solution = None
        self.best_fitness = float('inf')
        self.fitness_history = []
        
        # Initialize algorithm-specific components
        if algorithm_type == AlgorithmType.ACO:
            self.pheromone = np.ones((problem_size, problem_size))
        elif algorithm_type == AlgorithmType.GA:
            self.population = self._initialize_population()
            
    def validate_inputs(self, problem_size: int, max_iterations: int) -> None:
        """Validate input parameters"""
        if problem_size < 2:
            raise ValueError("Problem size must be at least 2")
        if max_iterations < 1:
            raise ValueError("Max iterations must be positive")

    def _initialize_population(self) -> List[List[int]]:
        """Initialize GA population"""
        population = []
        for _ in range(self.params.population_size):
            solution = list(range(self.problem_size))
            random.shuffle(solution)
            population.append(solution)
        return population

    def _crossover(self, parent1: List[int], parent2: List[int]) -> List[int]:
        """Order Crossover (OX) operator for GA"""
        size = len(parent1)
        # Select crossover points
        point1, point2 = sorted(random.sample(range(size), 2))
        
        # Create child using ordered crossover
        child = [-1] * size
        # Copy segment from parent1
        child[point1:point2] = parent1[point1:point2]
        
        # Fill remaining positions from parent2
        remaining = [x for x in parent2 if x not in child[point1:point2]]
        j = 0
        for i in range(size):
            if child[i] == -1:
                child[i] = remaining[j]
                j += 1
                
        return child

    def _mutate(self, solution: List[int]) -> List[int]:
        """Swap mutation for GA"""
        if random.random() < self.params.mutation_rate:
            i, j = random.sample(range(len(solution)), 2)
            solution[i], solution[j] = solution[j], solution[i]
        return solution

    def _construct_ant_solution(self, distances: np.ndarray) -> List[int]:
        """Construct solution for single ant in ACO"""
        solution = [random.randint(0, self.problem_size-1)]
        unvisited = set(range(self.problem_size)) - {solution[0]}
        
        while unvisited:
            current = solution[-1]
            # Calculate probabilities for next city
            probs = []
            for next_city in unvisited:
                pheromone = self.pheromone[current][next_city]
                distance = distances[current][next_city]
                if distance == 0:
                    distance = 1e-10
                prob = (pheromone ** self.params.alpha) * \
                       ((1.0 / distance) ** self.params.beta)
                probs.append((next_city, prob))
            
            # Select next city using roulette wheel
            total = sum(p[1] for p in probs)
            r = random.random() * total
            cum_sum = 0
            for city, prob in probs:
                cum_sum += prob
                if cum_sum >= r:
                    solution.append(city)
                    unvisited.remove(city)
                    break
                    
        return solution

    def _update_pheromone(self, solutions: List[Tuple[List[int], float]]) -> None:
        """Update pheromone trails in ACO"""
        # Evaporation
        self.pheromone *= (1 - self.params.pheromone_decay)
        
        # Add new pheromone
        for solution, distance in solutions:
            if distance == 0:
                continue
            deposit = 1.0 / distance
            for i in range(len(solution)-1):
                self.pheromone[solution[i]][solution[i+1]] += deposit
                self.pheromone[solution[i+1]][solution[i]] += deposit

    def _get_neighbor(self, solution: List[int]) -> List[int]:
        """Generate neighbor for SA using 2-opt move"""
        neighbor = solution.copy()
        i, j = sorted(random.sample(range(len(solution)), 2))
        neighbor[i:j+1] = reversed(neighbor[i:j+1])
        return neighbor

    def optimize(self, distances: np.ndarray) -> Tuple[List[int], float]:
        """Run optimization using selected algorithm
        
        Args:
            distances: Distance matrix
            
        Returns:
            Tuple of best route and its length
        """
        if self.algorithm_type == AlgorithmType.GA:
            return self._optimize_ga(distances)
        elif self.algorithm_type == AlgorithmType.ACO:
            return self._optimize_aco(distances)
        else:
            return self._optimize_sa(distances)

    def _optimize_ga(self, distances: np.ndarray) -> Tuple[List[int], float]:
        """Genetic Algorithm optimization"""
        for iteration in range(self.max_iterations):
            # Tournament selection
            new_population = []
            for _ in range(self.params.population_size // 2):
                parent1 = min(random.sample(self.population, 3),
                            key=lambda x: self._calculate_distance(x, distances))
                parent2 = min(random.sample(self.population, 3),
                            key=lambda x: self._calculate_distance(x, distances))
                
                # Crossover
                if random.random() < self.params.crossover_rate:
                    child1 = self._crossover(parent1, parent2)
                    child2 = self._crossover(parent2, parent1)
                else:
                    child1, child2 = parent1.copy(), parent2.copy()
                
                # Mutation
                child1 = self._mutate(child1)
                child2 = self._mutate(child2)
                
                new_population.extend([child1, child2])
            
            self.population = new_population
            
            # Update best solution
            current_best = min(self.population,
                             key=lambda x: self._calculate_distance(x, distances))
            current_distance = self._calculate_distance(current_best, distances)
            
            if current_distance < self.best_fitness:
                self.best_fitness = current_distance
                self.best_solution = current_best.copy()
            
            self.fitness_history.append(self.best_fitness)
            
        return self.best_solution, self.best_fitness

    def _optimize_aco(self, distances: np.ndarray) -> Tuple[List[int], float]:
        """Ant Colony Optimization"""
        for iteration in range(self.max_iterations):
            # Construct solutions
            ant_solutions = []
            for _ in range(self.params.n_ants):
                solution = self._construct_ant_solution(distances)
                distance = self._calculate_distance(solution, distances)
                ant_solutions.append((solution, distance))
            
            # Update best solution
            iteration_best = min(ant_solutions, key=lambda x: x[1])
            if iteration_best[1] < self.best_fitness:
                self.best_fitness = iteration_best[1]
                self.best_solution = iteration_best[0].copy()
            
            # Update pheromone trails
            self._update_pheromone(ant_solutions)
            self.fitness_history.append(self.best_fitness)
            
        return self.best_solution, self.best_fitness

    def _optimize_sa(self, distances: np.ndarray) -> Tuple[List[int], float]:
        """Simulated Annealing optimization"""
        # Initialize solution
        current_solution = list(range(self.problem_size))
        random.shuffle(current_solution)
        current_distance = self._calculate_distance(current_solution, distances)
        
        temperature = self.params.initial_temp
        
        while temperature > self.params.min_temp:
            for _ in range(self.problem_size):
                # Generate neighbor
                neighbor = self._get_neighbor(current_solution)
                neighbor_distance = self._calculate_distance(neighbor, distances)
                
                # Accept or reject new solution
                delta = neighbor_distance - current_distance
                if delta < 0 or random.random() < math.exp(-delta / temperature):
                    current_solution = neighbor
                    current_distance = neighbor_distance
                    
                    if current_distance < self.best_fitness:
                        self.best_fitness = current_distance
                        self.best_solution = current_solution.copy()
            
            temperature *= self.params.cooling_rate
            self.fitness_history.append(self.best_fitness)
            
        return self.best_solution, self.best_fitness

    def _calculate_distance(self, solution: List[int], distances: np.ndarray) -> float:
        """Calculate total distance of a route"""
        return sum(distances[solution[i]][solution[i+1]]
                  for i in range(len(solution)-1)) + \
               distances[solution[-1]][solution[0]]

    def plot_convergence(self) -> None:
        """Plot convergence history"""
        import matplotlib.pyplot as plt
        plt.figure(figsize=(10, 6))
        plt.plot(self.fitness_history)
        plt.title('Convergence History')
        plt.xlabel('Iteration')
        plt.ylabel('Best Fitness')
        plt.yscale('log')
        plt.grid(True)
        plt.show()
```
