# GoM CO₂ Storage Portfolio Optimization
## Optimization Methodology Guide — Version 1.0

**Muhammad Zulqarnain, Ph.D.**

Petroleum Engineering · CO₂ Geological Storage · Reservoir Simulation · Data Science & Optimization

---

## 1. Purpose

The Gulf of Mexico CO₂ Storage Portfolio Optimization platform is an engineering decision-support application designed to evaluate portfolios of depleted offshore oil and gas sands for potential geological CO₂ storage.

The application extends site-level storage screening into a portfolio-selection problem by integrating:

- Storage capacity
- CO₂ injectivity
- Offshore water depth
- Reservoir depth
- Storage-site complexity
- Distance from representative Gulf Coast pipeline hubs
- Pipeline tortuosity
- User-defined portfolio requirements

Mixed-Integer Linear Programming (MILP) is used to identify combinations of storage sites that satisfy specified storage-capacity and injectivity targets while minimizing a comparative Development Index.

The methodology is intended for regional screening and decision support. It is not intended to replace site-specific reservoir characterization, simulation, geomechanical analysis, pipeline routing, FEED studies, or detailed project economics.

---

# 2. Storage-Site Database

The optimization framework is built on a preprocessed Gulf of Mexico sandstone reservoir database derived from U.S. Bureau of Ocean Energy Management (BOEM) information.

The optimization dataset contains:

> **8,194 candidate offshore locations**

Each location can contain one or more depleted oil and gas sands.

The optimization framework uses location-level engineering attributes including:

- Geographic coordinates
- Number of candidate storage sands
- Offshore water depth
- Reservoir depth
- Estimated CO₂ storage capacity
- Estimated CO₂ injectivity
- Scenario-dependent engineering properties

The application operates at the **storage-location level**, rather than optimizing individual sands independently.

The underlying database represents depleted oil and gas sands. Regional saline formations are not included in the current optimization framework.

---

# 3. Engineering Scenarios

Storage capacity and injectivity are scenario-dependent quantities.

The application allows the user to select engineering assumptions before constructing the optimization problem.

## 3.1 Water-Production Scenario

Four water-production scenarios are available:

- 0×
- 0.5×
- 1×
- 2×

The selected scenario determines the storage-capacity estimate assigned to each candidate location.

Conceptually:

> Cᵢ = storage capacity of location *i* under the selected water-production scenario

The default application scenario is:

> **1× water production**

---

## 3.2 Fracture-Gradient Scenario

Three fracture-gradient scenarios are available:

- 0.7 psi/ft
- 0.8 psi/ft
- 0.9 psi/ft

The selected fracture-gradient assumption affects the allowable injection-pressure envelope and therefore the estimated site injectivity.

Conceptually:

> Iᵢ = maximum injectivity of location *i* under the selected fracture-gradient scenario

The default application scenario is:

> **0.8 psi/ft**

---

# 4. Candidate-Site Screening

Before optimization, the application constructs an eligible candidate set.

A location must contain valid values for both:

- Storage capacity under the selected water-production scenario
- Injectivity under the selected fracture-gradient scenario

Locations with missing injectivity are not imputed. They are excluded from the optimization candidate set.

The user can then impose additional minimum site-level thresholds:

> Cᵢ ≥ Cmin

and

> Iᵢ ≥ Imin

where:

- Cmin = minimum acceptable site storage capacity
- Imin = minimum acceptable site injectivity

The default screening thresholds are:

> **Minimum site capacity = 5 Mt**

> **Minimum site injectivity = 0.5 Mt/yr**

These screening criteria prevent the optimizer from selecting very small storage opportunities solely because they have favorable spatial or development characteristics.

The thresholds are user-adjustable and therefore form part of the decision scenario rather than permanent exclusions from the underlying database.

---

# 5. Representative Gulf Coast Pipeline Hubs

The spatial component of the optimization framework evaluates candidate storage locations relative to 11 representative Gulf Coast pipeline hubs:

1. Corpus Christi / Harbor Island, Texas
2. Freeport, Texas
3. Texas City / Galveston, Texas
4. Port Arthur / Sabine, Texas
5. Cameron / Calcasieu Pass, Louisiana
6. Intracoastal City, Louisiana
7. Morgan City / Berwick, Louisiana
8. Cocodrie / Terrebonne, Louisiana
9. Port Fourchon, Louisiana
10. Venice / Plaquemines, Louisiana
11. Pascagoula, Mississippi

These locations provide representative coastal connection points for comparative infrastructure screening.

They should not be interpreted as confirmation of existing or planned CO₂ export terminals.

---

# 6. Hub-to-Site Distance

For every candidate storage location, the geodesic distance to each representative Gulf Coast hub is calculated from geographic coordinates.

Let:

> Dgeo,i,h = geodesic distance between storage location *i* and hub *h*

The resulting distance represents the approximate straight-line geographic separation between the two locations.

Straight-line distance does not represent a practical offshore pipeline route. Actual pipelines must account for routing constraints, infrastructure corridors, bathymetry, seabed conditions, environmental restrictions, existing facilities, and other engineering considerations.

A pipeline tortuosity factor is therefore applied.

---

# 7. Pipeline Tortuosity

Conceptual pipeline length is estimated as:

> Dpipeline,i,h = TF × Dgeo,i,h

where:

- Dpipeline,i,h = conceptual routed pipeline distance
- Dgeo,i,h = geodesic hub-to-site distance
- TF = pipeline tortuosity factor

The default value is:

> **TF = 1.20**

Thus, a candidate location located 100 miles geodesically from a selected hub would have a conceptual pipeline distance of:

> 1.20 × 100 = 120 miles

The tortuosity factor is user-adjustable.

This treatment provides a screening-level representation of routing complexity. It does not constitute engineered pipeline routing.

---

# 8. Development Index

The optimization objective uses a dimensionless **Development Index (DI)** rather than a monetary cost estimate.

For candidate location *i* connected to hub *h*:

> DIᵢ,h = DIpipeline + DIoffshore + DIsubsurface + DIcomplexity

The four components represent:

1. Pipeline-distance burden
2. Offshore-development burden
3. Subsurface-development burden
4. Storage-site complexity

The Development Index is a comparative screening metric. It should not be interpreted as project CapEx, OpEx, levelized storage cost, or any other monetary project metric.

---

# 9. Pipeline Component

The pipeline component increases linearly with conceptual pipeline length.

The adopted scaling is:

> DIpipeline = Dpipeline × (5 / 200)

or equivalently:

> DIpipeline = 0.025 × Dpipeline

where pipeline distance is expressed in miles.

Under this convention:

> **200 routed pipeline miles = 5 Development Index units**

Examples:

| Conceptual Pipeline Length | Pipeline Index |
|---:|---:|
| 50 mi | 1.25 |
| 100 mi | 2.50 |
| 200 mi | 5.00 |
| 300 mi | 7.50 |

This coefficient is a comparative weighting factor and does not represent a pipeline construction cost per mile.

---

# 10. Offshore Development Component

Offshore development burden increases with water depth.

A piecewise-linear penalty is used so that increasingly deep offshore environments receive progressively larger incremental penalties.

Let:

> WD = water depth in feet

The offshore component is:

```text
DIoffshore =

0.40 × min(WD, 50) / 100

+ 0.50 × clip(WD - 50, 0, 150) / 100

+ 0.60 × clip(WD - 200, 0, 300) / 100

+ 0.75 × clip(WD - 500, 0, 500) / 100

+ 0.90 × clip(WD - 1000, 0, 2000) / 100

+ 1.00 × max(WD - 3000, 0) / 100
```

where `clip(x, a, b)` restricts the value of *x* to the interval *a* to *b*.

The piecewise formulation represents increasing offshore-development burden as water depth increases.

It is intended to distinguish relative development difficulty across shelf, deepwater, and ultra-deepwater settings without claiming project-specific offshore development costs.

---

# 11. Subsurface Development Component

Reservoir depth influences drilling requirements, well construction, pressure management, completion design, and overall subsurface development complexity.

Let:

> RD = mean reservoir depth in feet subsea

The subsurface component is:

```text
DIsubsurface =

0.10 × min(RD, 5000) / 1000

+ 0.25 × clip(RD - 5000, 0, 3000) / 1000

+ 0.40 × clip(RD - 8000, 0, 4000) / 1000

+ 0.60 × clip(RD - 12000, 0, 4000) / 1000

+ 0.80 × max(RD - 16000, 0) / 1000
```

The incremental penalty increases with reservoir depth.

Reservoir depth is treated as **subsea reservoir depth**. Water depth is therefore not added again to the reservoir-depth term.

This avoids double counting the offshore water column.

---

# 12. Storage-Site Complexity Component

A single geographic location may contain multiple candidate storage sands.

Locations containing more sands may offer additional storage opportunities, but they may also require more complex characterization, completion planning, surveillance, pressure management, and development strategies.

A moderate nonlinear complexity penalty is therefore applied:

> DIcomplexity = 0.30 × sqrt(max(Nsand - 1, 0))

where:

- Nsand = number of candidate sands at the location

For a single-sand location:

> DIcomplexity = 0

The square-root relationship increases the penalty as sand count increases while avoiding an excessively strong linear penalty for locations containing many sands.

---

# 13. Total Development Index

For each candidate site and selected pipeline hub:

> DIᵢ,h = DIpipeline,i,h + DIoffshore,i + DIsubsurface,i + DIcomplexity,i

Only the pipeline component changes when the same storage location is evaluated relative to a different hub.

The offshore, subsurface, and complexity components are intrinsic to the candidate storage location.

This structure allows the same candidate database to be efficiently evaluated under multiple infrastructure scenarios.

---

# 14. MILP Portfolio Formulation

After candidate screening and Development Index calculation, the portfolio-selection problem is formulated as a Binary Mixed-Integer Linear Program.

## 14.1 Candidate Set

Let:

> i ∈ S

where **S** is the set of eligible storage locations remaining after engineering screening.

---

## 14.2 Decision Variable

For each candidate location:

> xᵢ ∈ {0,1}

with:

> xᵢ = 1 if storage location *i* is selected

> xᵢ = 0 otherwise

The binary decision variable converts the storage-selection problem into a discrete portfolio optimization problem.

---

# 15. Objective Function

The optimization minimizes the combined Development Index of the selected portfolio.

> **Minimize Σ DIᵢ,h xᵢ**

The optimizer therefore seeks the combination of storage sites that satisfies all portfolio requirements with the lowest total comparative development burden.

Because the Development Index is additive, the objective remains linear.

---

# 16. Storage-Capacity Constraint

The combined storage capacity of the selected portfolio must meet or exceed the user-defined portfolio target:

> **Σ Cᵢ xᵢ ≥ Ctarget**

where:

- Cᵢ = storage capacity of candidate location *i*
- Ctarget = required portfolio storage capacity

The inequality allows the optimized portfolio to exceed the specified target when discrete site selection makes exact equality impractical.

---

# 17. Injectivity Constraint

The combined injectivity of the selected portfolio must meet or exceed the required portfolio injection rate:

> **Σ Iᵢ xᵢ ≥ Itarget**

where:

- Iᵢ = estimated injectivity of candidate location *i*
- Itarget = required portfolio injectivity

This prevents the optimizer from selecting a portfolio with sufficient total storage volume but inadequate injection capability.

---

# 18. Maximum-Site Constraint

The number of selected storage locations is limited by:

> **Σ xᵢ ≤ Nmax**

where:

- Nmax = maximum number of storage sites permitted in the portfolio

This constraint represents the practical preference to limit the number of independently developed offshore storage locations.

It also forces the optimization to consider the tradeoff between individual site quality and overall portfolio complexity.

---

# 19. Complete Optimization Problem

The complete optimization model can therefore be written as:

```text
Minimize:

    Σ DIᵢ,h xᵢ


Subject to:

    Σ Cᵢ xᵢ ≥ Ctarget

    Σ Iᵢ xᵢ ≥ Itarget

    Σ xᵢ ≤ Nmax

    xᵢ ∈ {0,1}
```

This is a Binary Mixed-Integer Linear Programming problem.

---

# 20. Optimization Engine

The mathematical model is implemented using:

> **Pyomo**

Pyomo provides the algebraic modeling framework used to define:

- Candidate-site sets
- Site parameters
- Binary decision variables
- Objective function
- Portfolio constraints

The model is solved using:

> **HiGHS**

HiGHS provides the MILP solution engine used by the application.

The optimization model is rebuilt using the current user-defined engineering assumptions whenever the user executes a new portfolio optimization.

---

# 21. Feasibility

Not every combination of user-defined targets is feasible.

For example, infeasibility may occur when:

- The capacity target is too large
- The injectivity target is too high
- The maximum number of sites is too restrictive
- Minimum site-level screening thresholds are too restrictive
- A selected engineering scenario substantially reduces the eligible candidate set

When the MILP is infeasible, the application does not force or approximate a solution.

Instead, it reports that no feasible portfolio was identified under the specified constraints.

The user can then modify the engineering or portfolio assumptions.

---

# 22. All-Hub Comparative Optimization

In addition to optimizing a portfolio for a single selected pipeline hub, the application can evaluate all 11 representative Gulf Coast hubs.

The same optimization problem is solved independently for each hub.

For each hub *h*:

1. Geodesic site distances are retrieved.
2. The selected tortuosity factor is applied.
3. Pipeline Development Index values are recalculated.
4. Total site Development Index values are constructed.
5. The MILP is solved using the same portfolio constraints.
6. Portfolio-level results are recorded.

The following quantities are compared:

- Number of selected sites
- Total storage capacity
- Total injectivity
- Total conceptual pipeline mileage
- Total Development Index
- Optimization feasibility

All other assumptions remain fixed during the hub comparison.

This isolates the effect of the assumed Gulf Coast connection point on portfolio selection.

---

# 23. Interpretation of the Hub Comparison

The all-hub comparison should be interpreted as a set of **independent infrastructure scenarios**.

It does not represent simultaneous optimization of:

- Multiple CO₂ sources
- Multiple pipeline hubs
- Shared trunk pipelines
- Pipeline branching
- Offshore gathering networks
- CO₂ allocation between hubs
- Network flow

For a given set of assumptions, a lower Development Index indicates a lower combined comparative development burden under the methodology used by the application.

It does not establish that a particular hub is economically superior under detailed project evaluation.

---

# 24. Spatial Visualization

The optimized portfolio is displayed using an interactive geographic map.

The map includes:

- Representative Gulf Coast pipeline hubs
- Selected pipeline hub
- Eligible offshore storage locations
- MILP-selected storage locations
- Conceptual hub-to-site pipeline connections

The map is intended to provide spatial context for the optimization results.

Conceptual pipeline lines connect the selected hub directly to the optimized storage locations.

These lines are not engineered pipeline routes.

Reported pipeline mileage includes the selected tortuosity factor, whereas the displayed map connections provide a simplified geographic representation.

---

# 25. Portfolio Outputs

For an optimal solution, the application reports portfolio-level metrics including:

- Number of selected storage sites
- Combined storage capacity
- Combined injectivity
- Total conceptual pipeline mileage
- Total Development Index

Site-level results include:

- Field or location name
- Storage capacity
- Injectivity
- Pipeline length
- Water depth
- Reservoir depth
- Number of sands
- Development Index

The selected-site portfolio can also be exported for additional analysis.

---

# 26. Default Decision Scenario

The application initializes using the following screening and optimization assumptions:

| Parameter | Default |
|---|---:|
| Pipeline Hub | Intracoastal City |
| Water Scenario | 1× |
| Fracture Gradient | 0.8 psi/ft |
| Minimum Site Capacity | 5 Mt |
| Minimum Site Injectivity | 0.5 Mt/yr |
| Portfolio Capacity Target | 200 Mt |
| Portfolio Injectivity Target | 10 Mt/yr |
| Maximum Sites | 3 |
| Pipeline Tortuosity Factor | 1.20 |

These values provide a representative starting point for interacting with the optimization model.

They are not intended to represent a prescribed development scenario.

---

# 27. Engineering Interpretation

The optimization framework illustrates an important distinction between **site ranking** and **portfolio optimization**.

A conventional ranking approach evaluates individual storage locations independently.

Portfolio optimization instead asks:

> Which combination of storage locations best satisfies a defined system-level requirement?

A location that appears attractive when considered independently may not be part of the optimized portfolio if another combination of sites provides:

- Adequate capacity
- Adequate injectivity
- Shorter infrastructure connections
- Lower offshore-development burden
- Lower subsurface-development burden
- Lower combined Development Index

MILP therefore provides a systematic mechanism for evaluating discrete engineering tradeoffs across a large candidate set.

---

# 28. Screening-Level Nature of the Methodology

The optimization results should be interpreted within the resolution of the underlying regional database.

The methodology does not replace detailed evaluation of:

- Reservoir continuity
- Fault and fracture characterization
- Caprock integrity
- Geomechanical response
- Pressure interference
- CO₂ plume migration
- Well integrity
- Legacy well risk
- Detailed injectivity testing
- Dynamic reservoir simulation
- Pipeline hydraulics
- Pipeline routing
- Offshore facilities design
- Regulatory permitting
- Project economics

Candidate locations identified by the optimization should therefore be considered **screening-level opportunities for further evaluation**.

---

# 29. Development Index Limitations

The Development Index is deliberately dimensionless.

It provides a consistent objective function for comparative portfolio optimization but does not represent:

- Capital expenditure
- Operating expenditure
- Net present value
- Levelized cost of CO₂ storage
- Pipeline tariff
- Cost per tonne
- Commercial project ranking

The weighting functions used for pipeline distance, water depth, reservoir depth, and sand complexity are engineering screening surrogates.

A future project-specific implementation could replace these terms with calibrated economic cost functions without changing the fundamental MILP portfolio architecture.

---

# 30. Pipeline Modeling Limitations

Pipeline distance is based on:

> geodesic distance × tortuosity factor

This provides an efficient regional approximation but does not account explicitly for:

- Bathymetric routing
- Seafloor hazards
- Existing infrastructure
- Exclusion zones
- Environmental constraints
- Pipeline diameter
- CO₂ phase behavior
- Compression requirements
- Hydraulic pressure loss
- Shared pipeline infrastructure

Consequently, pipeline mileage should be interpreted as a comparative screening metric.

---

# 31. Storage Resource Scope

The optimization database represents depleted offshore oil and gas sands included in the underlying GoM screening dataset.

The results therefore do not represent the total geological CO₂ storage potential of the Gulf of Mexico.

In particular:

> **Regional saline formations are outside the current application scope.**

Saline formations may contain substantially larger storage resources than the depleted oil and gas sands represented in this application.

---

# 32. Potential Framework Extensions

The current application deliberately uses a relatively transparent MILP formulation so that the relationship between engineering assumptions and portfolio selection remains interpretable.

The same framework could subsequently be extended to include:

- Project-level CapEx and OpEx functions
- Existing offshore infrastructure
- Legacy-well remediation costs
- Maximum pipeline-distance constraints
- Storage-site risk penalties
- Source-to-sink CO₂ allocation
- Multiple CO₂ sources
- Shared pipeline networks
- Pipeline-capacity constraints
- Multi-period development scheduling
- Scenario optimization
- Robust optimization
- Stochastic optimization

These extensions would represent different optimization problems and are not included in Version 1.0 of the application.

---

# 33. Summary

The GoM CO₂ Storage Portfolio Optimization platform integrates regional subsurface screening with spatial infrastructure analysis and mathematical optimization.

The workflow can be summarized as:

```text
GoM Storage Database
        ↓
Engineering Scenario Selection
        ↓
Candidate-Site Screening
        ↓
Hub-to-Site Distance Calculation
        ↓
Development Index
        ↓
Binary MILP Portfolio Selection
        ↓
Spatial and Portfolio Decision Support
```

The central optimization question is:

> **Which combination of offshore storage locations can satisfy the required CO₂ storage capacity and injectivity while minimizing the comparative development burden and respecting a limit on the number of storage sites?**

The resulting framework provides a transparent bridge between **reservoir engineering screening, spatial infrastructure considerations, and operations-research methods** for regional CO₂ storage decision support.

---

## Live Application

**[Launch the GoM CO₂ Storage Portfolio Optimization App](https://gom-co2-portfolio-optimization.streamlit.app/)**

---

## Author

**Muhammad Zulqarnain, Ph.D.**

Petroleum Engineering · CO₂ Geological Storage · Reservoir Simulation · Data Science & Optimization