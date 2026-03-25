# Cross-Validation Under Regime Shifts

## Overview

Standard cross-validation (K-fold, temporal CV) assumes stationarity or treats all historical periods as equally informative about future deployment. When data arise from multiple regimes — periods with distinct data-generating processes — these procedures estimate the wrong generalization target, leading to poor hyperparameter selection.

This paper develops **regime-aware cross-validation**, a procedure that weights validation evidence by its relevance to the anticipated deployment distribution. The key insight: the data-generating process evolves continuously over time, and validation folds from periods whose distributions are closer to the deployment distribution should receive more weight. Discrete regimes are a tractable approximation to this continuous evolution, and the weighting over regimes can be derived from a model of the regime dynamics.

## Connection to Hamilton (1989) Regime-Switching Models

Hamilton's (1989) Markov-switching framework models economic time series as being governed by a latent state variable that follows a Markov chain. In each state (regime), the data-generating process has different parameters — for example, GDP growth behaves differently in recessions vs. expansions. The transition matrix **M** governs how the economy moves between states: M_{rs} = Pr(next regime = s | current regime = r).

This paper borrows Hamilton's regime-switching machinery but applies it to a different problem: **cross-validation weighting**. Rather than using the Markov chain to model the data itself, we use it to determine how to weight validation evidence when selecting hyperparameters:

1. **Hamilton's use**: Estimate which regime the economy is in, and how parameters differ across regimes. The Markov chain is the *model of the data*.

2. **Our use**: Given observed regime boundaries, estimate the transition matrix M. Then use M to construct a probability distribution π over which historical regime the *next* deployment period will resemble. Weight CV validation folds by π. The Markov chain is a *tool for constructing validation weights*.

The connection is: if regimes are persistent (the current regime is informative about the next), then π is non-uniform, and regime-aware CV differs from — and improves upon — temporal CV, which implicitly uses uniform weights. When regimes are i.i.d. (no persistence), π is uniform and regime-aware CV reduces to standard temporal CV.

## Paper Structure

### Completed (Sections 1–4)
- **Section 1 — Problem statement**: Motivates the failure of standard CV under regime shifts
- **Section 2 — Deployment objectives**: Three targets (regime-specific, robust, mixture); paper focuses on regime-specific
- **Section 3 — Setup and goal**: Formal framework with regime-indexed distributions P_r on a common measurable space, training mixture Q, deployment distribution P, deployment risk R(λ; Q→P), and oracle hyperparameter λ*
- **Section 4 — Comparison to Temporal CV**: Shows temporal CV targets a historical average of next-block risks; identifies the persistent-regime setting as where the mismatch is consequential

### MAIN CONRIBUTION: To Write (Sections 5–7)

#### Section 5: Regime-Aware Cross-Validation (core contribution)

**5.1 — Motivation: continuously evolving distributions**
The true data-generating process is a continuously evolving distribution P_t, not a collection of discrete states. A "recovery period" is not a single frozen distribution but a trajectory through distribution space. Discrete regimes are a principled discretization — analogous to histograms approximating a smooth density — that is natural when we observe few time periods with many observations each.

**5.2 — General weighted-validation framework**
Define the regime-aware CV estimator with weights π on validation folds. Show that existing methods are special cases:
  - π uniform -> temporal CV (i.i.d. regimes)
  - π from exponential decay -> EWKCV (ad hoc, from recent literature)
  - π from Markov transition matrix -> regime-aware CV (main theoretical case)
  - π worst-case -> robust/DRO approach

**5.3 — Deriving π from a Markov regime process**
Model the regime sequence as a Markov chain with transition matrix M. Given the current regime, π is the conditional distribution over the next regime. Estimate M from observed regime transitions. This connects directly to Hamilton (1989).

**5.4 — Bias characterization (key result)**
Derive the explicit bias of temporal CV for the deployment risk and bound it in terms of the total variation distance(is this the right metric to use? tbd) between π and the uniform distribution. Show that regime-aware CV eliminates this bias when π is correctly specified. The result is interpretable: the more persistent the regime dynamics, the larger the bias of temporal CV, and the greater the improvement from regime-aware weighting.

**5.5 — Consistency (key result)**
Prove that the hyperparameter selected by regime-aware CV converges to the oracle hyperparameter under standard regularity conditions. Show that temporal CV converges to the wrong oracle when regimes are persistent.

#### Section 6: Discussion / Future Work
- Full taxonomy: i.i.d. -> temporal CV; persistent -> regime-aware; adversarial -> robust
- Latent regimes: soft labels from HMMs, structural break detection as upstream step
- Continuous drift: nonparametric weighting via distributional distances (MMD, Wasserstein spaces?)
- Sensitivity to misspecification of π
- Finite-sample risk bounds

#### Section 7: Empirical Demonstration
- Simulation study under persistent regimes
- Possible application to credit risk data with macroeconomic regime structure

## Key References

| Paper | Relevance |
|---|---|
| Hamilton (1989) | Markov regime-switching models — foundation for deriving π |
| Bai & Perron (1998) | Structural break detection — upstream tool for identifying regime boundaries |
| Sugiyama et al. (2007, JMLR) | Importance Weighted CV — reweights training observations under covariate shift |
| Elliott & Timmermann (2005) | Regime-switching forecast combination — closest in spirit (regime-driven weights, but for combining forecasts, not CV) |
| Pesaran & Timmermann (2007) | Estimation window selection under structural breaks |
| Han, Huang & Wang (2024, ICML) | Adaptive rolling window for model selection under temporal shift — distribution-free, no regime model |
| EWKCV (2024) | Ad hoc exponential weighting of temporal CV folds |
| HyperTime (2024) | Worst-case hyperparameter selection under temporal shift |
| Dahlhaus (1997) | Locally stationary processes — theoretical foundation for continuous drift |

## Open Questions

1. How sensitive is the procedure to misspecification of π?
2. How many regime transitions are needed to estimate M reliably?
3. Can crude regime labels still improve on uniform weighting?
4. Finite-sample risk bounds?

## Next Steps

- [ ] Write Section 5.2: general weighted-validation framework + population target
- [ ] Write Section 5.4: bias characterization (key theoretical result)
- [ ] Write Section 5.3: Markov chain derivation of π
- [ ] Prove consistency (Section 5.5)
- [ ] Write Section 6: discussion/future work
- [ ] Design simulation study
- [ ] Add bibliography with proper BibTeX citations
