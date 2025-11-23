# Ambulance Base Placement for Emergency Response Optimisation

This project studies how to place ambulance bases in a Tier-2 Indian city so that emergency response times are as low as possible, under a limit on how many bases can be opened.

We model the city of Mysore as a set of demand zones, compute realistic road-network travel times from each zone to each candidate ambulance base, and then solve a p-median style facility location problem using Mixed Integer Linear Programming (MILP).

---

## Problem Statement

Emergency medical response depends heavily on how well ambulances are positioned in the city. Poorly located bases lead to:

- long travel distances  
- uneven coverage across neighbourhoods  
- delays that can be the difference between life and death  

The central question we address is:

> **Where should ambulance bases be located, and which zones should they serve, to minimise emergency response time under a limit on the number of bases?**

We focus on Mysore (Mysuru), divide the city into zones, and assume ambulances respond from fixed bases located at hospitals.

---

## Approach

### 1. City Zoning and Demand Representation

- The city is divided into a grid of rectangular **demand zones**.  
- Each zone is represented by its **centroid**.  
- Zones that do not intersect the Mysore city polygon are removed, leaving 23 valid zones.  
- All zones are currently treated with **equal demand weight** (this can be replaced by real call data later).

### 2. Candidate Base Locations

- Candidate ambulance base locations are chosen from hospital locations or network nodes inside Mysore.  
- Each candidate base is assumed to have equal capability and a fixed opening cost.

### 3. Travel Time Matrix

- We first tried using OSMnx with a constant speed, but that only gave distance based times.
- We then switched to **OpenRouteService (ORS)** to compute realistic driving times:
  - For each zone centroid `i` and candidate base `j`, we query ORS for driving time.
  - This produces a matrix \( T_{ij} \) of **travel times in minutes**.

### 4. Optimisation Model (p-median style MILP)

We use a p-median style facility location model:

- **Decision variables**
  - \( y_j \in \{0,1\} \): 1 if base \( j \) is opened, 0 otherwise  
  - \( x_{ij} \in \{0,1\} \): 1 if zone \( i \) is assigned to base \( j \), 0 otherwise  

- **Objective (for fixed K)**  
  Minimise total demand-weighted travel time:
  \[
  \min \sum_i \sum_j w_i \, T_{ij} \, x_{ij}
  \]

- **Constraints**
  - Assignment: each zone is assigned to exactly one base  
    \[
    \sum_j x_{ij} = 1 \quad \forall i
    \]
  - Consistency: zones can only be assigned to open bases  
    \[
    x_{ij} \le y_j \quad \forall i,j
    \]
  - Budget: at most \( K \) bases may be open  
    \[
    \sum_j y_j \le K
    \]

For each value of \(K\) we solve this MILP using PuLP + CBC and compute:

- demand-weighted average response time  
- maximum response time  
- number of open bases

### 5. Penalised Objective to Choose K

Opening more bases reduces response time but increases cost. To balance this, we define:

\[
J(K) = \text{avg\_time}(K) + \lambda \cdot \text{cost}(K)
\]

- `avg_time(K)` is the demand-weighted average response time for that K  
- `cost(K)` is proportional to the number of open bases (each base has a fixed cost)  
- \( \lambda \) (minutes per crore) controls how strongly we penalise cost  

For \( \lambda = 0.1 \), we evaluate \( J(K) \) for all tested K and pick the K where \( J(K) \) is minimal.  
In our experiments, this penalised objective is minimised at **K = 13**, which we treat as the recommended number of bases.

We then visualise:

- the 13 selected bases  
- zone-to-base assignments  
- resulting average and worst case response times  


# How to run

git clone https://github.com/Deepaksaai/Yes-Kings-opti-project.git
cd Yes-Kings-opti-project

python -m venv .venv
source .venv/bin/activate   # on Windows: .venv\Scripts\activate

pip install -r requirements.txt

Sign up for an API key on OpenRouteService and set it in your code, for example:

```text
import openrouteservice as ors

ORS_API_KEY = "YOUR_KEY_HERE"  # replace with your key inside quotes
client = ors.Client(key=ORS_API_KEY)

```
Start jupyter notebook and run final.ipynb







