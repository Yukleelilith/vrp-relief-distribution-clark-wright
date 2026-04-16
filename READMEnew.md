# Clark-Wright VRP — Relief Goods Distribution (Peñaranda)

**Application of Graph Theory in the Optimization of Relief Goods Distribution — BSM CS 4B**

## Project Overview

This project applies graph theory and combinatorial optimization to model and solve the Vehicle Routing Problem (VRP) for relief goods distribution across 51 barangays in the Municipality of Peñaranda. We implement the Clark-Wright Savings Algorithm to generate optimized multi-stop delivery routes from the Municipal Hall (depot), minimizing total travel time while respecting vehicle capacity constraints of 2,500 units per truck.

## How to Run

```
python clark_wright_vrp.py
```

## Output Files Generated

### Results (`vrp_results.txt`)

| File | Description |
|------|-------------|
| `vrp_results.txt` | Full route listing with load, travel time, and summary statistics |

## Folder Structure

- `clark_wright_vrp.py` — Main solver: matrix builder, savings algorithm, and report printer
- `vrp_results.txt` — Generated output with optimized routes and summary
