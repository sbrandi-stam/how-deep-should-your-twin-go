> Evidence audit for the [Twin Depth Selector](README.md). This document is the source of truth for every claim shown in the tool: verified figures, evidence labels, excluded sources with reasons, and open verifications. Licensed under CC BY 4.0.

# Model Depth Selector — Content Hypothesis v3

**Status:** evidence-audited working draft (v3 = external audit merged with 3 substantiated amendments; see Changelog)
**Application anchor:** supervisory control closing the digital-twin loop through building-level setpoint optimization; local PID loops remain unchanged.

The selector describes the **minimum model depth likely to preserve decision validity**, not a universal ranking of model quality. FDD is treated separately because observability and fault localization often require more component detail than supervisory optimization.

## Evidence labels

- **[FIELD-OP]** occupied, operational building demonstration
- **[FIELD-TEST]** real test building or short pilot
- **[LAB]** controlled laboratory experiment
- **[SIM]** simulation study
- **[REVIEW]** peer-reviewed review
- **[INDUSTRY]** industry-authored or industry-commissioned analysis
- **[HYP]** engineering hypothesis or inference not directly established by the cited source
- **[VERIFIED]** claim checked against the primary text (abstract or full text) during drafting

Quantitative figures always carry a label, scope, comparator, and — where material — test duration.

---

## 0. System type selector

These axes do not change the model-depth definitions. They modify the emphasis and confidence of the recommendations.

### Emission system

**High-inertia radiant / TABS**
- Thermal mass and long time constants make anticipative control especially relevant.
- A comparative review, citing two primary studies, reports that activating the thermal storage capacity in slabs results in **30–50% lower peak load** and allows smaller conditioning equipment [REVIEW citing primary studies, R14 — VERIFIED in review text]. Scope: peak-load reduction from mass activation, **not** a direct estimate of MPC value.
- Selector effect [HYP]: raise the importance of Envelope & Zones depth and of explicit heat-emission dynamics.
- Wording: slow response / long time constants.

**Hydronic radiators**
- Intermediate dynamics; potentially important interaction between supply temperature, flow, valve authority, and emitter output.
- Selector effect [HYP]: baseline guidance applies; flag distribution and return-temperature effects where the supervisory controller can influence them.

**All-air / VAV / fan-coil dominant**
- Faster zone and terminal dynamics generally reduce the value of very long thermal prediction horizons.
- Selector effect [HYP]: measured occupancy and near-term disturbance information become relatively more important.
- Design hypothesis: the current evidence set has no controlled same-building comparison across emission-system types.

### Generation

**Heat pump**
- COP varies with source/sink temperatures and temperature lift. The importance of modeling this variation grows when supervisory actions materially alter supply temperature, storage temperature, or operating lift [SIM, R13; FIELD-TEST, R12].
- In a 7-day real-building demonstration, MPC reduced electricity cost by ~40% (procurement price from 0.36 to 0.12 EUR/kWh) and improved the Grid Support Coefficient by 13%, while average COP fell from 7.6 to 6.0 as average temperature lift increased [FIELD-TEST, 7 days, R12 — VERIFIED]. The authors conclude MPC can enhance heat pump operation **with simple component models**, provided the system offers flexibility [R12 — VERIFIED].
- The lesson displayed in the tool: the price signal can outweigh the efficiency loss — but only a model that sees the COP–lift coupling can tell you when it does not.
- Selector effect: HVAC L1 flagged **high risk for load shifting** when the chosen actions materially change temperature lift. Do not state that a constant-COP model is universally unusable.

**Condensing boiler**
- Return temperature is an important efficiency lever.
- Selector effect [HYP; reference needed]: increase the relevance of distribution and return-temperature modeling when those variables are controllable. Add a primary source before publishing as evidence-backed.

**Hybrid / multi-carrier**
- The supervisory value often lies in coordinating devices and energy carriers.
- Selector effect [HYP]: model breadth and correct coupling constraints may matter more than high detail in any single component. Architectural hypothesis, not a demonstrated quantitative result.

---

## 1. Envelope & Zones

### L1 — Single-zone, low-order grey-box

**Pros**
- Low calibration and computational burden relative to multi-zone or white-box models.
- Can be sufficient for price-driven load shifting when zone diversity is limited and comfort can be represented by an aggregate state.

**Cons**
- Zone-level comfort and diversity are hidden.
- Aggregation error increases as internal gains, orientations, schedules, and terminal behavior diverge.

**Evidence and counterweight**
- R2 supports the proposition that the controller order needed for best MPC performance can exceed the order commonly used in compact grey-/black-box models [SIM, R2 — VERIFIED in abstract].
- The separate ~50% white-box vs grey-box energy comparison in a 12-zone office belongs to **R18, not R2**; present with the original authors' caveat that results depended on model and plant assumptions [SIM, R18 — citation pending verification].

**FDD note**
- Usually insufficient for component-level localization; may still support aggregate residual-based anomaly detection.

### L2 — Multi-zone grey-box

**Pros**
- Makes zone comfort and diversity explicit.
- Multi-zone identification tractable through structured or zone-wise estimation [SIM, R3 — VERIFIED in abstract].

**Cons**
- Sensor and identification effort scale with the number and diversity of zones.
- Parameter identifiability and maintenance become practical constraints.

**Field-test counterweight**
- In a real thermally activated test environment, no significant correlation was found between prediction accuracy and control performance; explicit representation of heat-emission physics mattered more than uniformly increasing model detail [FIELD-TEST, R4 — VERIFIED in abstract]. Scope: small test environment, not long-term occupied deployment.

### L3 — Calibrated white-box BEM or reduced physics model

**Pros**
- Most defensible when retrofit or scenario analysis changes the underlying physics and historical data no longer represent the future system (reduced physics models and validated surrogates may also be suitable).
- Higher-fidelity simulated control performance when parameters, boundary conditions, and forecasts are sufficiently accurate [SIM, R2; SIM, R18].

**Cons**
- High calibration and governance burden; generally requires reduction, co-simulation, or a surrogate for real-time optimization; needs ongoing recalibration.

**FDD note**
- Can serve as whole-building reference model; component-level FDD still depends on instrumentation, residual design, and fault identifiability.

### Cross-cutting sensor caveat

The model does not automatically know whether a temperature sensor represents the occupied-zone air volume. A sensor exposed to direct solar radiation or local drafts feeds a point measurement the model interprets as a zone-average state. A simulation-based methodology on a real dwelling attributed ~7% energy savings to thermostat sensing-location optimization, payback under 2.5 years [SIM/case, R15 — abstract attributes the saving to sensor location]. **Reviewer objection pending:** an external audit claims the tested scenario also changed the setpoint to 21 °C, which would dilute the attribution; this has not been confirmed in the full text. The claim stands as per abstract until the full-text check resolves it either way (see To-verify).

---

## 2. HVAC — Generation & Distribution

### L1 — Static efficiency maps
Examples: constant COP or boiler efficiency, rated capacity limits.

**Pros**
- Supports linear or convex optimization.
- May be adequate when efficiency is nearly flat over the operating range actually used by the controller.

**Cons**
- Does not represent part-load behavior, source/sink temperature effects, or changing temperature lift.
- For heat-pump load shifting: high risk when optimization changes supply or storage temperatures [FIELD-TEST, R12; SIM, R13].

**FDD note**
- Insufficient for component-level diagnosis; aggregate efficiency monitoring may still be possible.

### L2 — Quasi-dynamic component models
Examples: temperature-dependent COP, simplified emitter dynamics, adaptive linear predictors.

**Pros**
- Captures the couplings most likely to affect supervisory decisions without a full plant model.
- R4 indicates explicit heat-emission physics can matter more than uniformly increasing model order [FIELD-TEST, R4].
- Adaptive COP predictors retain tractable optimization while representing operating-point dependence [SIM, R16].

**Cons**
- Requires manufacturer data, field data, or dedicated identification tests.
- Model validity must be checked over the range explored by the optimizer.

### L3 — White-box hydronic network
Examples: pressure–flow relations, valve authority, pumps, emitters, distribution losses.

**Pros**
- Enables commissioning analysis; may expose distribution constraints and losses hidden at L1–L2.
- Provides observability for some distribution-level FDD tasks.

**Cons**
- As-built hydraulic data often incomplete; high maintenance burden.
- Supervisory optimization may not exploit this resolution unless it controls pump head, flow allocation, supply/return temperatures, or valve coordination.

**Evidence status**
- R5 (hydronic optimization ~10 kWh/m²·year, payback <5 years in typical retrofits) is authored by Danfoss employees and describes Danfoss-commissioned research [INDUSTRY, peer-reviewed venue, R5 — provenance VERIFIED via eceee author listing]. Retained with explicit label; never described as independent; not used as the sole quantitative justification for L3.

**Open question (displayed in tool)**
Does hydronic detail belong in the online twin, in an offline commissioning model, or in both? The answer depends on which variables the supervisory layer can actuate and which faults it must localize.

---

## 3. Thermal Storage

### L1 — One-node capacity with losses

**Pros**
- Convex and simple to identify.
- In laboratory MPC tests on electric water heaters, one-node MPC reduced energy cost by **12.3–23.2%** vs thermostatic control under time-of-use tariffs [LAB, R7 — VERIFIED in primary text].

**Cons**
- Cannot represent vertical stratification; state of charge can be biased when usable energy depends strongly on temperature layers.

### L2 — Multi-node stratified model

**Pros**
- Same laboratory study: three-node MPC achieved **31.2–42.5%** cost reduction vs thermostatic control — roughly double the one-node benefit [LAB, R7 — VERIFIED in primary text].

**Counterweight**
- Experimental validation of simplified tank models: two-node models did not consistently outperform optimized one-node models; differences grew with stratification level [LAB/model-validation, R8 — VERIFIED in abstract]. Note: R8 evaluates predictive model performance, not closed-loop MPC benefit.
- The tank's physical stratification level, not the designer's ambition, decides whether this level pays.

**Scope caveat (displayed)**
- R7: residential electric water heaters. R8: water-tank configurations under DHW profiles. Extrapolation to central plant buffers is plausible but not directly demonstrated.

### L3 — High-resolution / CFD

**Pros**
- Tank design, inlet geometry, diffuser design, offline stratification studies.

**Cons**
- Usually outside the needs of the online supervisory loop.
- Evidence wording: no source in the current evidence set demonstrates incremental closed-loop control benefit over L2.

---

## 4. Electrical Systems — PV and Battery

### L1 — Linear battery model
Examples: energy integrator, constant charge/discharge efficiency, power and SOC bounds.

**Pros**
- LP/MILP friendly; common in energy-management formulations.

**Cons**
- Can misrepresent the revenue–degradation trade-off when dispatch is aggressive: low-fidelity control over-commits the battery relative to a high-fidelity physics-based benchmark [SIM, R6 — VERIFIED in primary text].

### L2 — Nonlinear efficiency and degradation-aware penalty

**Pros**
- A low-complexity MPC with a well-designed terminal penalty reproduced the behavior of the high-fidelity controller [SIM, R6 — VERIFIED in primary text]. Practical lesson: the optimization needs the right degradation signal more than a full electrochemical state model.

**Cons**
- Penalty calibration requires degradation data or conservative assumptions.
- R6 concerns stationary batteries in frequency-regulation and energy markets, not a building HEMS; the transfer is an inference [HYP on transfer].

### L3 — Electrochemical–thermal model

**Pros**
- Cell-level safety, thermal constraints, degradation-aware market participation at scale.

**Cons**
- Usually excessive for building EMS unless the control objective directly depends on electrochemical or thermal states; the current evidence set does not demonstrate incremental building-EMS control benefit over a well-designed L2 model.

---

## 5. Occupancy

### L1 — Fixed schedules / standard profiles

**Pros**
- Low cost, easy deployment.
- Consistent with the fit-for-purpose principle when occupancy uncertainty has little influence on the control decision [R1 — VERIFIED in abstract].

**Cons**
- Can become a major source of performance error in intermittently occupied or diverse-use buildings.

**Wording note**
- R1 supports parsimony and fit-for-purpose selection; sufficiency of fixed schedules is a conditional design judgment, not a proven general claim.

### L2 — Measured occupancy as information
Use cases: reactive setback, disturbance input, robust-control signal.

**Pros**
- The strongest evidence concerns measured presence rather than long-horizon behavioral prediction.
- 20-month single-family field test: **17.6% cooling-energy savings in the 2023 cooling season** under occupancy-based thermostat control [FIELD-OP, R9 — primary paper VERIFIED: Energy and Buildings 328, 115161].
- Same study: imperfect sensing (false negatives), comfort problems during sleep, and extended high-demand periods on summer afternoons with potential grid implications; the savings figure is always shown with these limitations [FIELD-OP, R9 — VERIFIED].
- 27-home community simulation: savings ~**1–20%**, strongly dependent on occupancy pattern and setback strategy [SIM, R10].

**Cons**
- Sensor cost, false positives/negatives, privacy and governance.
- Context-dependent evidence; not a universal saving.

### L3 — Behavioral / individual prediction

**Pros**
- May add value when pre-conditioning matters, comfort bands are flexible, and prediction quality is sufficient.

**Cons and evidence**
- Predictive-vs-reactive comparisons depend strongly on comfort constraints: at equal temperature bounds, reactive occupancy control outperformed predictive; predictive occupancy cases with medium temperature bounds consumed approximately **20.8% less energy than the reactive case** while maintaining comfort [SIM, R11 — VERIFIED in primary arXiv text; reinstated after audit query].
- Many earlier studies assume ideal future occupancy; in a virtual testbed, realistic prediction errors degraded the comfort–energy trade-off [SIM/virtual testbed, R17].

**Tool stance**
- Measure first; predict only when prediction changes a valuable anticipative decision and the comfort-risk trade-off is explicit. The R11 result is displayed with its condition: the wider bands, not the prediction alone, carry much of the gain.

---

## Changelog v3 (vs external audit)

1. **R11 — 20.8% reinstated.** The audit flagged it as unverified; the figure was verified directly in the primary arXiv text during drafting, with its condition (medium temperature bounds). Audit claim rejected.
2. **R14 — 30–50% restored with correct attribution.** The claim exists in the review text (citing its primary refs [16,17]); scoped as peak-load reduction from mass activation, not MPC value. The audit's substitute figure (10–30% radiant vs all-air) moved to To-verify: no trace provided.
3. **R15 — audit weakening suspended.** The abstract attributes the 7% saving to sensor-location optimization; the audit's claim of a simultaneous 21 °C setpoint change is not confirmed. Displayed per abstract, flagged pending full-text check.
4. Verification statuses updated: R5 provenance (Danfoss authorship) confirmed via eceee page; R9 primary paper confirmed (E&B 328, 115161) including limitations; R12 details confirmed (7 days, 40% cost, COP 7.6→6.0, simple-models conclusion).

## References

- **R1** — Gaetani I., Hoes P.-J., Hensen J.L.M. (2016). Occupant behavior in building energy simulation: Towards a fit-for-purpose modeling strategy. *Energy and Buildings* 121, 188–204. https://doi.org/10.1016/j.enbuild.2016.03.038
- **R2** — Picard D., Drgoňa J., Kvasnica M., Helsen L. (2017). Impact of the controller model complexity on model predictive control performance for buildings. *Energy and Buildings* 152, 739–751. https://doi.org/10.1016/j.enbuild.2017.07.027 [SIM]
- **R3** — Arroyo J., Spiessens F., Helsen L. (2020). Identification of multi-zone grey-box building models for use in model predictive control. *Journal of Building Performance Simulation* 13(4), 472–486. https://doi.org/10.1080/19401493.2020.1770861 [SIM]
- **R4** — Arroyo J., Spiessens F., Helsen L. (2022). Comparison of Model Complexities in Optimal Control Tested in a Real Thermally Activated Building System. *Buildings* 12(5), 539. https://doi.org/10.3390/buildings12050539 [FIELD-TEST]
- **R5** — Kolb S. (Danfoss), Osojnik M. (Danfoss Trata) (2019). Costs and benefits of optimizing hydronic performance of water-based heating and cooling systems. *eceee Summer Study Proceedings*, peer-reviewed. [INDUSTRY — provenance verified]
- **R6** — Cao Y., Lee S.B., Subramanian V.R., Zavala V.M. (2020). Multiscale model predictive control of battery systems for frequency regulation markets using physics-based models. *Journal of Process Control*. https://doi.org/10.1016/j.jprocont.2020.04.001 [SIM]
- **R7** — Buechler E., Goldin D., Rajagopal R. (2024). Improving the Load Flexibility of Stratified Electric Water Heaters: Design and Experimental Validation of MPC Strategies. *IEEE Transactions on Smart Grid* 15, 3613–3623. https://doi.org/10.1109/TSG.2024.3366116 [LAB — journal citation pending verification; figures verified in arXiv:2312.04102]
- **R8** — Hernández Hernández C., Péan T., Salom J. (2026). Simplified Grey-Box Models for Stratified Water Tanks in HVAC Systems. *CLIMA 2025 Proceedings*, LNCE 788, 741–751. https://doi.org/10.1007/978-3-032-10546-2_69 [LAB/model-validation]
- **R9** — Pang Z., Guo M., O'Neill Z., Smith-Cortez B., Yang Z., Liu M., Dong B. (2025). Long-term field testing of the accuracy and HVAC energy savings potential of occupancy presence sensors in a single-family home. *Energy and Buildings* 328, 115161. https://doi.org/10.1016/j.enbuild.2024.115161 [FIELD-OP — verified]
- **R10** — Munankarmi P., Maguire J., Jin X. (2022). Occupancy-Based Controls for an All-Electric Residential Community in a Cold Climate. *IEEE PES General Meeting 2022*. [SIM — citation pending verification; figures verified via OSTI 1908955]
- **R11** — Gluck J. et al. (2017). A Systematic Approach for Exploring Tradeoffs in Predictive HVAC Control Systems for Buildings. arXiv:1705.02058 [SIM — 20.8% figure verified in primary text]
- **R12** — Tomás L., Lämmle M., Pfafferott J. (2025). Demonstration and Evaluation of MPC for a Real-World Heat Pump System in a Commercial Low-Energy Building. *Energies* 18(6), 1434. https://doi.org/10.3390/en18061434 [FIELD-TEST, 7 days — verified]
- **R13** — Pieper H., Ommen T., Jensen J.K., Elmegaard B., Markussen W.B. (2020). Comparison of COP estimation methods for large-scale heat pumps used in energy planning. *Energy* 205, 117994. https://doi.org/10.1016/j.energy.2020.117994 [SIM; district-heating scale]
- **R14** — Hesaraki A., Huda N. (2022). A comparative review on the application of radiant low-temperature heating and high-temperature cooling. *Sustainable Energy Technologies and Assessments* 49, 101661. https://doi.org/10.1016/j.seta.2021.101661 [REVIEW — 30–50% peak-load claim verified in review text, citing its refs 16–17]
- **R15** — Seri F., Arnesano M., Keane M.M., Revel G.M. (2021). Temperature Sensing Optimization for Home Thermostat Retrofit. *Sensors* 21(11), 3685. https://doi.org/10.3390/s21113685 [SIM/case]
- **R16** — Rastegarpour S., Scattolini R., Ferrarini L. (2021). Performance improvement of an air-to-water heat pump through linear time-varying MPC with adaptive COP predictor. *Journal of Process Control* 99, 69–78. https://doi.org/10.1016/j.jprocont.2021.01.006 [SIM]
- **R17** — Yang T., Sangogboye F.C., Arendt K. et al. (2021). The Impact of Occupancy Prediction Accuracy on the Performance of MPC in Buildings. *Building Simulation 2021 Proceedings*. https://doi.org/10.26868/25222708.2021.30571 [SIM/virtual testbed]
- **R18** — Picard D., Sourbron M., Jorissen F., Váňa Z., Cigler J., Ferkl L., Helsen L. (2016). Comparison of MPC Performance Using Grey-Box and White-Box Controller Models of a Multi-zone Office Building. *4th International High Performance Buildings Conference at Purdue*, Paper 203. [SIM — citation pending verification]

## To-verify before publication

1. **R7** journal citation (IEEE TSG 15, 3613–3623) — figures already verified in the arXiv version; confirm the journal reference.
2. **R18** Purdue 2016 paper — confirm existence, authors, and the ~50% figure with its caveats in the primary text.
3. **R14** audit's 10–30% radiant-vs-all-air figure — locate in review full text before any use.
4. **R15** full text — check whether the 7% scenario changed the setpoint alongside sensor location; apply the audit's weakening only if confirmed.
5. **R10** IEEE PESGM citation details.
6. Condensing-boiler return-temperature claim — needs a primary source.
7. Mahdavi & Tahmasebi (2015) — still excluded until the specific conclusion is read in the primary text.

## Excluded

- ITG Dresden hydronic figures (vendor-channel provenance, methods and funding unverified).
- Community-battery 42% figure (microgrid generator context, not building EMS).

## Declared evidence gaps (tool footer)

- No controlled same-building comparison quantifies MPC benefit across TABS, radiators, and all-air systems; emission-system modifiers rest on juxtaposition of separate studies [HYP].
- Storage evidence comes from residential water heaters and DHW tank tests, not central plant buffers.
- R2/R18 (simulation favors depth) and R4 (field test decouples prediction accuracy from control performance) point in different directions under different conditions; presented as context dependence, not a resolved contradiction.
- Battery evidence (R6) comes from market-participation studies rather than building HEMS.
- Several selector modifiers are engineering hypotheses, explicitly labelled [HYP].
