"""
=============================================================================
 APPLICATION OF GRAPH THEORY IN THE OPTIMIZATION OF RELIEF GOODS DISTRIBUTION
 Vehicle Capacity: 2,500 units
=============================================================================
"""

import math
from io import StringIO
import sys

# =============================================================================
# SECTION 1: INPUT YOUR DATA HERE
# =============================================================================

VEHICLE_CAPACITY = 2500

# ---------------------------------------------------------------------------
# 1B. DEMAND PER BARANGAY (nodes 1 to 51)
#     Node 0 = Municipal Hall (depot), demand is always 0
# ---------------------------------------------------------------------------
demand = {
    0:  0,
    1:  400,  2:  951,  3:  428,  4:  1020,  5:  23,
    6:  989,  7:  150,  8:  0,  9:  50,  10: 2483,
    11: 125,  12: 350,  13: 250,  14: 159,  15: 400,
    16: 300,  17: 217,  18: 1388,  19: 145,  20: 435,
    21: 503,  22: 23,  23: 2287,  24: 259,  25: 2396,
    26: 0,  27: 295,  28: 942,  29: 262,  30: 350,
    31: 2257,  32: 378,  33: 0,  34: 1200,  35: 350,
    36: 450,  37: 235,  38: 65,  39: 1295,  40: 120,
    41: 745,  42: 0,  43: 721,  44: 45,  45: 474,
    46: 2400,  47: 194,  48: 15,  49: 1223,  50: 2,
    51: 300,
}

# ---------------------------------------------------------------------------
# 1C. EDGE LIST — every direct road connection
# ---------------------------------------------------------------------------
edge_list = [
    # Depot (node 0) to nearby barangays
    (0, 1,  6.8), (0, 10,  3.8), (0, 23,  6.4), (0, 20,  2.6),
    (0, 51,  11.4), (0, 31,  3.8), (0, 18,  3), (0, 16,  5.2),
    (0, 5, 10), (0, 25, 5.4),

    # Cluster A (nodes 1–10)
    (1, 10,  5.2), (2, 4,  3.2), (2, 6,  2),
    (2, 35,  3.6), (2, 39,  1.3), (2, 45,  1.1),
    (3, 4,  9.9), (3, 29,  1.8), (3, 32, 1.7),
    (4, 35,  1.6), (4, 29, 9.3),
    (5, 16,  4.3), (5, 17,  1.4), (5, 19, 4.4),
    (5, 40,  2.6), (5, 51,  3.4),
    (6, 7,  2.6), (6, 17, 5.8),
    (6, 35, 4), (7, 40, 5.6),
    (8, 26, 3), (8, 33, 1.5),
    (9, 44, 1.5), (10, 23, 2.6), (10, 31, 7), (10, 12, 8.4),

    # Cluster B (nodes 11–20)
    (11,22, 1.9), (11,48, 2.4),
    (12,21, 1.6), (12,37, 1.2),
    (12,38, 1.3), (13,39, 3),
    (13, 45, 2), (14, 32, 1.5),
    (14, 34, 2.58), (15, 41, 1.4),
    (15, 47, 1.8), (16,21, 2.2),
    (16, 27, 0.9), (17, 40, 3.72),
    (17, 27, 3.9), (18, 38, 0.8),
    (19,22, 6.2), (19,48, 2.2),
    (19,49, 3.6), (20,31, 1.9), (20,49, 3.4),

    # Cluster C (nodes 21–30)
    (21,27, 2.4), (21,37, 1.3),
    (21,38, 0.8), (21,47, 2),
    (22,51, 8), (23,25, 7.4),
    (23,36, 12.6), (24,25, 3.6),
    (24,31, 10.2), (24,44, 3.8),
    (26,42, 2), (28,30, 4.4),
    (28,50, 2.4), (29,32, 2),
    (30,35, 5.2),

    # Cluster D (nodes 31–40)
    (31,36, 3), (31,49, 3.3),
    (33,50, 1.5), (36,44, 1.4),
    (37,47, 1.8), (37,38, 0.9),
    (39,45, 2.2), (39,46, 4.6),
    (40,42, 4.4), (40,51, 3.2),

    # Cluster E (nodes 41–51)
    (41,47, 0.42), (42,51, 4.4),
    (43,46, 1.6), (43,47, 2),
    (46,47, 1.5), (48,49, 1.5),
    (44,45, 3.2), (44,51, 5.7),
    (45,46, 2.8), (45,51, 4.3),
    (46,47, 3.5), (46,50, 4.8),
    (47,48, 2.9), (47,49, 3.7),
    (48,49, 2.5), (49,50, 3.1),
    (50,51, 2.6),

]

coordinates = {}  # GPS coords not needed since we have edge list


# =============================================================================
# SECTION 2: MATRIX BUILDER
# =============================================================================

def build_travel_time_matrix(n_nodes, edges, coords):
    INF = 99999.0
    matrix = [[INF] * n_nodes for _ in range(n_nodes)]
    for i in range(n_nodes):
        matrix[i][i] = 0.0
    for (u, v, t) in edges:
        if t < matrix[u][v]:
            matrix[u][v] = t
            matrix[v][u] = t
    for k in range(n_nodes):
        for i in range(n_nodes):
            for j in range(n_nodes):
                if matrix[i][k] + matrix[k][j] < matrix[i][j]:
                    matrix[i][j] = matrix[i][k] + matrix[k][j]
    return matrix


# =============================================================================
# SECTION 3: CLARK-WRIGHT SAVINGS ALGORITHM
# =============================================================================

def compute_savings(n_nodes, travel_time):
    savings = []
    for i in range(1, n_nodes):
        for j in range(i + 1, n_nodes):
            s = travel_time[0][i] + travel_time[0][j] - travel_time[i][j]
            savings.append((s, i, j))
    savings.sort(reverse=True)
    return savings


def clark_wright(n_nodes, demand, travel_time, capacity):
    routes = {i: [0, i, 0] for i in range(1, n_nodes)}
    node_route = {i: i for i in range(1, n_nodes)}
    savings = compute_savings(n_nodes, travel_time)

    for (s, i, j) in savings:
        if s <= 0:
            break

        ri = node_route.get(i)
        rj = node_route.get(j)
        if ri is None or rj is None or ri == rj:
            continue

        route_i = routes[ri]
        route_j = routes[rj]

        i_at_end   = route_i[-2] == i
        i_at_start = route_i[1]  == i
        j_at_start = route_j[1]  == j
        j_at_end   = route_j[-2] == j

        merged = None
        if i_at_end   and j_at_start: merged = route_i[:-1] + route_j[1:]
        elif j_at_end and i_at_start: merged = route_j[:-1] + route_i[1:]
        elif i_at_start and j_at_end: merged = route_j[:-1] + route_i[1:]
        elif j_at_start and i_at_end: merged = route_i[:-1] + route_j[1:]
        if merged is None:
            continue

        load = sum(demand[n] for n in merged if n != 0)
        if load > capacity:
            continue

        routes[ri] = merged
        del routes[rj]
        for node in merged:
            if node != 0:
                node_route[node] = ri

    return list(routes.values())


# =============================================================================
# SECTION 4: REPORTING
# =============================================================================

def route_travel_time(route, travel_time):
    return round(sum(travel_time[route[k]][route[k+1]] for k in range(len(route)-1)), 1)

def route_load(route, demand):
    return sum(demand[n] for n in route if n != 0)

def nodes_served(route):
    return [n for n in route if n != 0]

def print_results(routes, demand, travel_time):
    sep = "=" * 75
    thin = "-" * 75
    print(f"\n{sep}")
    print("  CLARK-WRIGHT SAVINGS ALGORITHM — OPTIMIZED RELIEF DISTRIBUTION")
    print("  City of Malolos | Vehicle Capacity: 2,500 units")
    print(sep)
    print(f"  {'Route':<38} {'Nodes Served':<18} {'Load':>6}  {'Time (min)':>10}")
    print(thin)

    total_time = 0.0
    total_load = 0
    for idx, route in enumerate(sorted(routes, key=lambda r: r[1]), 1):
        route_str  = "-".join(str(n) for n in route)
        served     = nodes_served(route)
        served_str = ", ".join(str(n) for n in served)
        tt         = route_travel_time(route, travel_time)
        load       = route_load(route, demand)
        total_time += tt
        total_load += load
        print(f"  Route {idx}: {route_str:<30} {served_str:<18} {load:>6}  {tt:>10.1f}")

    print(thin)
    print(f"  {'':50} {'TOTAL':>6}  {round(total_time,1):>10.1f}")
    print(f"  {'':50} {total_load:>6}  {'':>10}")
    print(sep)
    print(f"\n  Summary:")
    print(f"    Total routes (trucks used)  : {len(routes)}")
    print(f"    Total barangays served       : {sum(len(nodes_served(r)) for r in routes)}")
    print(f"    Total relief goods delivered : {total_load:,} units")
    print(f"    Total travel time            : {round(total_time,1)} minutes")
    print(f"    Average time per route       : {round(total_time/len(routes),1)} minutes")
    print(sep)

def print_savings_table(savings, top_n=15):
    print(f"\n  TOP {top_n} SAVINGS PAIRS (Clark-Wright)")
    print(f"  {'Rank':<6} {'Node i':>8} {'Node j':>8} {'Saving (min)':>14}")
    print("  " + "-"*40)
    for rank, (s, i, j) in enumerate(savings[:top_n], 1):
        print(f"  {rank:<6} {i:>8} {j:>8} {round(s,2):>14.2f}")


# =============================================================================
# SECTION 5: MAIN
# =============================================================================

if __name__ == "__main__":
    N = 52  # node 0 (municipal hall) + nodes 1–51 (barangays)

    print("\nBuilding travel time matrix")
    travel_time = build_travel_time_matrix(N, edge_list, coordinates)

    print("Running Clark-Wright Savings Algorithm")
    routes = clark_wright(N, demand, travel_time, VEHICLE_CAPACITY)

    savings = compute_savings(N, travel_time)
    print_savings_table(savings, top_n=15)
    print_results(routes, demand, travel_time)

    with open("vrp_results.txt", "w") as f:
        buf = StringIO()
        old = sys.stdout; sys.stdout = buf
        print_savings_table(savings, top_n=15)
        print_results(routes, demand, travel_time)
        sys.stdout = old
        f.write(buf.getvalue())

    print("\n  Results saved to: vrp_results.txt")
