# RNeat

> An R implementation of the NEAT (NeuroEvolution of Augmenting Topologies) algorithm for evolving artificial neural networks through genetic algorithms.

---

## Overview

RNeat is an R package implementing the NEAT algorithm, a neuroevolution approach for generating artificial neural networks with variable topology. It combines genetic algorithms with topology evolution to solve control and classification problems by adaptively building network structure over generations.

---

## Features

- **Genetic Algorithm Framework** — Population-based evolution with selection, crossover, and mutation
- **Topology Evolution** — Networks grow and adapt connections over time
- **Node Innovation** — Add new neurons to network structure
- **Connection Innovation** — Add new connections with historical tracking
- **Speciation** — Group similar genomes to preserve diverse solutions
- **Fitness Tracking** — Monitor convergence and population statistics
- **Visualization** — Plot network topologies and evolution metrics
- **Configuration Control** — Customize mutation rates, population size, parameters

---

## Getting Started

### Requirements

- **R 3.5+**
- **igraph** — Network graph manipulation
- **ggplot2** — Visualization (optional)
- **tibble** — Data frame utilities

### Installation

```r
# Install from GitHub
devtools::install_github("s4lt3d/RNeat")

# Or install from source
install.packages("path/to/RNeat", repos = NULL, type = "source")
```

---

## Quick Start

```r
library(RNeat)

# Create NEAT configuration
config <- create_neat_config(
  population_size = 100,
  input_nodes = 2,
  output_nodes = 1,
  generations = 50
)

# Initialize population
population <- initialize_population(config)

# Run evolution
result <- evolve_population(
  population = population,
  fitness_func = your_fitness_function,
  config = config,
  generations = 50
)

# Get best genome and network
best_genome <- result$best_genome
best_network <- genome_to_network(best_genome)
```

---

## Core Concepts

### Genome Structure
```
├── Nodes
│   ├── Input nodes
│   ├── Hidden nodes (evolved)
│   └── Output nodes
└── Connections
    ├── Weight values
    ├── Enabled/disabled status
    └── Innovation numbers
```

### Evolution Process

```
Initial Population
    ↓
Evaluate Fitness
    ↓
Speciation (Group Similar Genomes)
    ↓
Selection & Reproduction
    ├── Crossover (Combine genomes)
    ├── Mutation
    │   ├── Weight mutation
    │   ├── Add node
    │   ├── Add connection
    │   └── Toggle connection
    └── Create Offspring
    ↓
Next Generation
```

---

## Configuration

Key parameters in `create_neat_config()`:

```r
config <- create_neat_config(
  # Network structure
  input_nodes = 2,
  output_nodes = 1,
  initial_hidden_nodes = 0,

  # Population
  population_size = 100,
  generations = 100,

  # Mutation rates
  add_node_rate = 0.01,
  add_connection_rate = 0.05,
  weight_mutation_rate = 0.8,
  weight_mutation_power = 0.5,

  # Speciation
  compatibility_threshold = 3.0,
  excess_coefficient = 1.0,
  disjoint_coefficient = 1.0,
  weight_coefficient = 0.4
)
```

---

## Usage Examples

### Simple Classification Problem

```r
# XOR problem
fitness_xor <- function(genome, config) {
  network <- genome_to_network(genome)

  inputs <- list(c(0, 0), c(0, 1), c(1, 0), c(1, 1))
  targets <- c(0, 1, 1, 0)

  error <- 0
  for (i in seq_along(inputs)) {
    output <- evaluate_network(network, inputs[[i]])
    error <- error + (output[1] - targets[i])^2
  }

  return(4.0 - error)  # Higher is better
}

# Run evolution
result <- evolve_population(
  population,
  fitness_xor,
  config,
  generations = 50
)
```

### Training on Dataset

```r
# Load your data
data <- read.csv("training_data.csv")
X <- as.matrix(data[, 1:4])  # features
y <- data$target

fitness_custom <- function(genome, config) {
  network <- genome_to_network(genome)

  predictions <- apply(X, 1, function(x) {
    evaluate_network(network, x)[1]
  })

  error <- mean((predictions - y)^2)
  return(1.0 / (1.0 + error))
}

result <- evolve_population(population, fitness_custom, config, 100)
```

---

## Project Structure

```
├── R/
│   ├── config.R          — Configuration management
│   ├── genome.R          — Genome representation
│   ├── network.R         — Network evaluation
│   ├── evolution.R       — Genetic operators
│   ├── speciation.R      — Speciation logic
│   └── visualization.R   — Plotting functions
├── data/
│   └── example_data.csv  — Sample dataset
├── examples/
│   ├── xor_problem.R
│   └── classification.R
├── man/                  — Function documentation
├── tests/                — Unit tests
└── README.md
```

---

## Network Evaluation

Evaluate a genome on inputs:

```r
# Convert genome to network
network <- genome_to_network(genome)

# Evaluate on single input
input <- c(0.5, 0.7)
output <- evaluate_network(network, input)

# Batch evaluation
inputs_list <- list(c(0, 0), c(0, 1), c(1, 0), c(1, 1))
outputs <- lapply(inputs_list, function(x) evaluate_network(network, x))
```

---

## Visualization

```r
# Plot network topology
plot_network(best_genome)

# Plot evolution metrics
plot_evolution_stats(result)

# Visualize speciation
plot_speciation(result)
```

---

## Advanced Features

### Custom Activation Functions
```r
# Default is tanh, customize with
config$activation_function <- function(x) 1 / (1 + exp(-x))  # sigmoid
```

### Saving and Loading

```r
# Save population
saveRDS(population, "population.rds")

# Load population
population <- readRDS("population.rds")
```

### Parallel Evolution

```r
# Evaluate fitness in parallel
library(parallel)
result <- evolve_population_parallel(
  population,
  fitness_func,
  config,
  num_cores = 4
)
```

---

## Performance Tips

- **Fitness Function** — Make it fast; it's called many times
- **Population Size** — Larger = slower but better convergence
- **Speciation** — Adjusts compatibility threshold based on diversity
- **Generation Limit** — Set reasonable stopping criteria

---

## Troubleshooting

### Networks Not Improving
- Increase mutation rates
- Adjust speciation threshold
- Verify fitness function is correct
- Check network can represent solution

### Slow Convergence
- Reduce population size for faster testing
- Increase add_node_rate and add_connection_rate
- Verify fitness function varies across population

### Memory Issues
- Reduce population size
- Use smaller hidden layer sizes
- Enable garbage collection between generations

---

## Algorithm Details

NEAT improves upon basic neuroevolution by:
1. **Historical tracking** — Tracks innovations across generations
2. **Topology evolution** — Grows network structure adaptively
3. **Speciation** — Protects innovation by isolating new topologies
4. **Meaningful mutation** — Network structure changes, not just weights

---

## References

- Stanley, K. O., & Miikkulainen, R. (2002). Evolving neural networks through augmenting topologies
- Original NEAT Paper: [http://nn.cs.utexas.edu/downloads/papers/stanley.cec02.pdf](http://nn.cs.utexas.edu/downloads/papers/stanley.cec02.pdf)

---

## License

Copyright © Walter Gordy
