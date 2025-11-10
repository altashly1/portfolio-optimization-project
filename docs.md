### Problem Statement

This project implements the robust portfolio selection framework from Goldfarb and Iyengar (2002). The goal is to compute an optimal asset allocation that is resilient to the statistical estimation errors inherent in historical financial data.

The problem is formulated as finding the **Robust Maximum Sharpe Ratio** portfolio, which maximizes the portfolio's return per unit of risk under the worst-possible realization of the model parameters.

---
### Given (Parameters)

The optimization problem is formulated using a set of parameters derived from a historical training dataset (2014-2023). A single-factor model, $r_i = \mu_i + V_i^T f + \epsilon_i$, is used, where all returns are defined as **annualized excess growth rates**.

* **Point Estimates:**
    * $\mu_0 \in \mathbb{R}^n$: The vector of estimated **mean annualized excess growth rates** (the intercepts).
    * $V_0 \in \mathbb{R}^n$: The vector of estimated **factor loadings** (the regression slopes) against the SPY factor.
    * $F \in \mathbb{R}$: The scalar variance of the factor's annualized excess growth rate, $\text{Var}(f)$. This is a point estimate; following the paper's main model (Sections 2-5), we assume $F$ is known and stable.
    * $\overline{D} \in \mathbb{R}^{n \times n}$: A diagonal matrix of the estimated residual variances ($s_i^2$). This serves as the "worst-case" bound for the $S_d$ set.

* **Uncertainty Sets (at $\omega=0.95$ confidence):**
    * $S_m$ (Mean Uncertainty): The set for the true mean $\mu$ is a "box" defined by the parameter $\gamma \in \mathbb{R}^n$:
        $$S_m = \{\mu \in \mathbb{R}^n : |\mu_i - \mu_{0,i}| \le \gamma_i, \quad i=1,...,n\}$$
    * $S_v$ (Factor Loading Uncertainty): The set for the true factor loading $V$ is an elliptical region defined by the parameter $\rho \in \mathbb{R}^n$:
        $$S_v = \{V \in \mathbb{R}^n : V = V_0 + W, \quad \|W_i\|_g \le \rho_i, \quad i=1,...,n\}$$
    * $S_d$ (Residual Variance Uncertainty): The set for the true residual variance $D$ is an interval. Our model's objective $\min \max \phi^T D \phi$ always selects the worst-case (upper) bound, $\overline{D}$.
        $$S_d = \{D : D = \text{diag}(d), \quad \underline{d}_i \le d_i \le \overline{d}_i, \quad i=1,...,n\}$$

---
### To Be Determined (Variables)

The optimization solves for the following variables:
* $\phi \in \mathbb{R}^n$: The vector of **portfolio weights** (allocations) for the $n=424$ assets, subject to the constraint $\phi \ge 0$ (no short selling).
* $\nu, \delta \in \mathbb{R}$: Auxiliary variables used to represent the worst-case factor and residual variance components, respectively.

---
### Objective

The conceptual objective is to solve the "max-min" problem for the Robust Sharpe Ratio over all defined uncertainty sets:

$$\max_{\phi} \min_{\mu \in S_m, V \in S_v, D \in S_d} \left\{ \frac{\mu^T \phi}{\sqrt{\phi^T(V^T F V + D)\phi}} \right\}$$

As shown in the paper, this transformation is possible by first assuming the optimal worst-case Sharpe ratio is **strictly positive**. The crucial step is realizing the objective function is **homogeneous** in $\phi$. This mathematical property allows the normalization constraint $\mathbf{1}^T \phi = 1$ to be dropped and replaced by a constraint on the numerator, $(\mu_0 - \gamma)^T \phi \ge 1$.

This transforms the fractional max-min problem into an equivalent convex optimization problem, where the constraints $\delta$ and $\nu$ represent the solved robust counterparts for $S_d$ and $S_v$:

$$\min_{\phi, \nu, \delta} \quad \nu^2 + \delta$$
$$\text{subject to:}$$
$$(\mu_0 - \gamma)^T \phi \ge 1 \quad\quad\quad\quad \text{(Robust counter-part for } S_m\text{)}$$
$$\phi^T \overline{D} \phi \le \delta \quad\quad\quad\quad \text{(Robust counter-part for } S_d\text{)}$$
$$\|F^{1/2} V_0 \phi\|_2 + \sqrt{\kappa} \rho^T \phi \le \nu \quad \text{(Robust counter-part for } S_v\text{, simplified model)}$$
$$\phi \ge 0$$