### Problem Statement

This project implements the robust portfolio selection framework from Goldfarb and Iyengar (2002). The goal is to compute an optimal asset allocation that is resilient to the statistical estimation errors inherent in historical financial data.

The problem is formulated as finding the **Robust Maximum Sharpe Ratio** portfolio. This formulation maximizes the portfolio's return per unit of risk under the worst-possible realization of the model parameters within their statistically determined uncertainty sets.

---
### Constants & Hyperparameters

These are the fixed, user-defined inputs that structure the model environment.

| Symbol | Definition | Value in Project |
| :--- | :--- | :--- |
| $n$ | **Number of Assets** in the investment universe. | 424 |
| $m$ | **Number of Factors** in the linear return model. | 1 (SPY index) |
| $p$ | **Number of Observations** (trading days) in the training set. | 2514 |
| $\omega$ | **Confidence Level** probability for constructing uncertainty sets. | 0.95 (95%) |
| $r_f$ | **Risk-Free Rate** (annualized) used to calculate excess returns. | 0.0363 (3.63%) |
| $\Delta t$| **Time Step**, the fraction of a year representing one trading day. | $1/252$ |

---
### Given (Parameters)

The optimization problem is formulated using parameters derived from a historical training dataset (2014-2023). We adopt the **factor model** from the paper, where the return of each asset $i$ is defined as:

$$r_i = \mu_i + V_i^T f + \epsilon_i$$

Here, $r_i$, $f$, and $\mu_i$ are all defined as **annualized excess growth rates**. The parameters for this model are not known with certainty and are estimated from the data, resulting in the following inputs for the robust problem:

* **Point Estimates (The "Center" of our Beliefs):**
    * $\mu_0 \in \mathbb{R}^n$: The vector of estimated mean annualized excess growth rates (the regression intercepts).
    * $V_0 \in \mathbb{R}^n$: The vector of estimated factor loadings (the regression slopes, as $m=1$).
    * $F \in \mathbb{R}$: The scalar variance of the factor's excess growth rate, $\text{Var}(f)$. Following the paper's main model, $F$ is assumed to be a known, stable point estimate.

* **Uncertainty Sets (The "Bounds" of our Beliefs):**
    * **$S_m$ (Mean Uncertainty):** The set for the true mean $\mu$. It is defined as a "box" centered at $\mu_0$ with a radius defined by $\gamma \in \mathbb{R}^n$:
        $$S_m = \{\mu \in \mathbb{R}^n : |\mu_i - \mu_{0,i}| \le \gamma_i, \quad \forall i=1, ..., n\}$$
    * **$S_v$ (Factor Loading Uncertainty):** The set for the true factor loading $V$. It is defined as an "elliptical" region centered at $V_0$ with a radius defined by $\rho \in \mathbb{R}^n$:
        $$S_v = \{V \in \mathbb{R}^n : V = V_0 + W, \quad \|W_i\|_g \le \rho_i, \quad \forall i=1, ..., n\}$$
    * **$S_d$ (Residual Variance Uncertainty):** The set for the true residual variance $D$. It is defined as an "interval" for which we use the worst-case (upper) bound, $\overline{D} \in \mathbb{R}^{n \times n}$:
        $$S_d = \{D : D = \text{diag}(d), \quad \underline{d}_i \le d_i \le \overline{d}_i, \quad \forall i=1, ..., n\}$$

* **Model Simplification Parameters:**
    * $\kappa \in \mathbb{R}$: A scalar parameter, $1/(p-1)$, used in the simplified SOCP constraint for $S_v$.

---
### Optimization Variables (To Be Determined)

These are the decision variables solved for by the optimizer.

| Symbol | Definition |
| :--- | :--- |
| $\phi$ | The $n \times 1$ vector of **portfolio weights** (allocations), constrained such that $\phi_i \ge 0, \forall i=1, ..., n$. |
| $\nu$ | A scalar auxiliary variable representing the worst-case **factor standard deviation**. |
| $\delta$ | A scalar auxiliary variable representing the worst-case **residual variance**. |

---
### Objective

The conceptual objective is to solve the max-min problem for the Robust Sharpe Ratio over all defined uncertainty sets:

$$\max_{\phi} \min_{\mu \in S_m, V \in S_v, D \in S_d} \left\{ \frac{\mu^T \phi}{\sqrt{\phi^T(V^T F V + D)\phi}} \right\}$$

By assuming the optimal worst-case Sharpe ratio is strictly positive and utilizing the homogeneity of the objective function, this is transformed into the following equivalent convex Second-Order Cone Program (SOCP).

The implemented objective is to **minimize the total worst-case variance** subject to the robust counterpart for each uncertainty set:

$$\min_{\phi, \nu, \delta} \quad \nu^2 + \delta$$
$$\text{subject to:}$$
$$(\mu_0 - \gamma)^T \phi \ge 1 \quad\quad\quad\quad \text{(Robust counter-part for } S_m\text{)}$$
$$\phi^T \overline{D} \phi \le \delta \quad\quad\quad\quad \text{(Robust counter-part for } S_d\text{)}$$
$$\|F^{1/2} V_0 \phi\|_2 + \sqrt{\kappa} \rho^T \phi \le \nu \quad \text{(Robust counter-part for } S_v\text{, simplified model)}$$
$$\phi_i \ge 0, \quad \forall i=1, ..., n$$