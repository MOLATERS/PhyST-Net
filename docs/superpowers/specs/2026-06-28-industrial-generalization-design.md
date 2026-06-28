# Industrial Generalization Design

## Objective

Reposition PhyST-Net from a wind-power-specific forecasting model to a general exogenous-driver-aware, graph-structured forecasting framework for constrained industrial time series. The target venue positioning is TKDE-style data engineering: the central contribution should be a reusable modeling interface for networked industrial forecasting rather than a single-domain wind application.

## Target Framing

The revised paper should describe PhyST-Net as a generic model for node-level forecasting on industrial networks with three input contracts:

- Historical node states, such as wind power, solar generation, or regional load.
- Historical and future exogenous drivers, such as weather, irradiance, temperature, demand indicators, control schedules, and calendar variables.
- Domain constraints, such as non-negativity, rated capacity, safety envelopes, or operating-range limits.

The preferred paper-level claim is:

> PhyST-Net provides a general ExG-aware constrained graph forecasting framework for industrial networked time series.

This framing preserves the current architecture while making the paper broader than wind farms. Energy-system datasets remain the first experimental instantiation because they naturally provide graph structure, known future drivers, and physical constraints.

## Dataset Plan

The recommended main experiment suite uses energy systems as a concrete industrial benchmark family and contains three domains with two forecasting horizons.

| Domain | Dataset | Purpose | Priority |
|---|---|---|---|
| Wind generation | SDWPF | Large-scale spatial wind-farm benchmark with rich turbine-level relations. | Required |
| Wind generation | HoT or Kelmarsh | Smaller real wind-farm validation with richer meteorological variables. | Required |
| Solar generation | NREL Solar Power Data for Integration Studies | Multi-site PV generation with irradiance and weather-driven dynamics. | Required |
| Electricity load | GEFCom2014 Load or Open Power System Data | Demand-side validation driven by temperature and calendar effects. | Required |
| Wind generation | EFJ | Additional external wind validation if page budget and compute allow. | Optional |

The main paper should avoid reintroducing ultra-short-term results. The short-term and medium-term settings better match the intended regime where future drivers, graph propagation, and constrained decoding jointly affect the forecast.

## Dataset Selection Criteria

Each added dataset should satisfy most of the following conditions:

- It contains multiple nodes, sites, zones, or locations that can form a graph.
- It provides external drivers known or forecastable over the prediction horizon.
- It supports physical or operational output constraints.
- It has enough history for chronological train, validation, and test splits.
- It can be converted into the existing sample format without changing the core model.

NREL Solar is the strongest solar candidate because irradiance and meteorological drivers align directly with K-AMC. GEFCom2014 or Open Power System Data is the strongest load candidate because temperature and calendar variables support demand-side exogenous conditioning.

## Architecture Mapping

The current modules can be described in domain-general terms:

- AGSP becomes continuous event-aware temporal tokenization. It targets non-stationary changes such as wind ramps, solar cloud transients, and load peak transitions.
- R-STMF becomes a spatial relation adapter. It can operate on turbine-distance graphs, PV-site geographic graphs, or regional load adjacency graphs.
- K-AMC becomes a future-driver conditioning interface. It ingests NWP for wind, irradiance and weather for solar, and temperature or calendar drivers for load.
- PI-DCH becomes a generic constrained decoding head. For wind and PV, it enforces non-negative generation and capacity-aware upper envelopes. For load, it should use non-negativity and, if defensible, a historical or rated demand envelope.

The manuscript should avoid describing PI-DCH only as a wind-capacity module. It should be introduced as a constraint operator whose concrete bounds are dataset-dependent.

## Minimum Experiment Matrix

The smallest credible TKDE-oriented main experiment matrix is:

| Domain | Dataset | Horizons |
|---|---|---|
| Wind | SDWPF | Short, Medium |
| Wind | HoT or Kelmarsh | Short, Medium |
| Solar | NREL Solar | Short, Medium |
| Load | GEFCom2014 or Open Power System Data | Short, Medium |

This gives four datasets, three industrial energy domains, and two forecasting horizons. It is broad enough to support the industrial generalization claim without exploding the page budget.

## Ablation Plan

The current SDWPF short-term ablation remains valid and should stay in the main paper because it has rich spatial relations and an appropriate prediction length. To support cross-domain generality, add one lightweight ablation on a non-wind task:

- Preferred: NREL Solar short-term ablation, focused on K-AMC and PI-DCH.
- Alternative: Load medium-term ablation, focused on AGSP and K-AMC.

The minimum additional ablation variants are:

- Full PhyST-Net.
- w/o AGSP.
- w/o R-STMF.
- w/o K-AMC.
- w/o PI-DCH or hard-clip-only decoding.

The main paper should report MAE and RMSE only. Compliance-rate metrics should not appear in the paper body.

## Baseline Plan

Use the same baseline family already used in the wind experiments where possible:

- TimeXer
- PatchTST
- iTransformer
- CrossFormer
- TiDE
- DAG
- TimeKAN if memory allows

For solar and load, the comparison should keep the same information contract: all models receive the same historical target window and available future exogenous drivers, either natively or through the existing exogenous adapter.

## Implementation Boundaries

This design does not require changing the core PhyST-Net architecture. The expected work is:

- Add dataset loaders for solar and load tasks.
- Map each dataset to the existing input tuple: historical target, historical covariates, future exogenous drivers, time marks, graph metadata, and target sequence.
- Add dataset-specific constraint bounds for PI-DCH.
- Create short and medium scripts for each added dataset.
- Run smoke tests before full training.

The implementation should first audit standardization to ensure that target and exogenous scalers are fitted on the training split only.

## Manuscript Revision Scope

If the new experiments are adopted, the manuscript should revise:

- Title or abstract wording from wind-power-specific language to constrained industrial forecasting.
- Problem definition to emphasize graph-structured industrial nodes rather than turbines only.
- Method descriptions that currently imply wind-only semantics.
- Experiment section title to "Experiments on Constrained Industrial Forecasting".
- Dataset table to include wind, solar, and load datasets.
- Discussion to explain domain transfer across industrial generation-side and demand-side forecasting.

The paper should not claim full cross-domain universality. The defensible claim is generality within constrained, exogenous-driver-aware industrial forecasting, validated first on energy-system networks.

## Success Criteria

The generalization plan is successful if:

- The revised experiment suite includes at least one solar dataset and one load dataset.
- PhyST-Net remains competitive on the added non-wind tasks.
- At least one non-wind ablation supports the value of K-AMC or PI-DCH.
- The final manuscript can state a coherent industrial forecasting contribution without relying on wind-only terminology.
