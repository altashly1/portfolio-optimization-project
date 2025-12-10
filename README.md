# Robust Portfolio Optimization via SOCP

**Course:** CHEME/SYSEN 6800 Computational Optimization  
**Team Members:** Abdulrahman Alswaidan, Grace Sun, Mason Wong, Rizu Zhao, Yue Zhang  

## 📌 Project Overview
This project implements a **Robust Optimization (RO)** framework for portfolio selection, addressing the sensitivity of classical Mean-Variance optimization to parameter estimation errors. 

Building on the methodology of **Goldfarb and Iyengar (2003)**, we reformulate the Robust Maximum Sharpe Ratio problem as a tractable **Second-Order Cone Program (SOCP)**. The model constructs uncertainty sets for expected returns, factor loadings, and residual variances based on statistical confidence regions, maximizing the worst-case Sharpe ratio.

### Key Features
* **Robust Formulation:** Explicitly models parameter uncertainty using ellipsoidal ($S_v$) and box ($S_m$) sets.
* **SOCP Implementation:** Solved efficiently using **Julia (JuMP)** and the **SCS** solver.
* **Empirical Testing:** Tested on 424 S&P 500 firms using 10 years of daily VWAP data (2014–2023).
* **Sensitivity Analysis:** Evaluates the impact of confidence levels ($\omega$) on portfolio concentration and risk-adjusted returns.

---

## 🛠️ Dependencies & Requirements

The project is implemented in **Julia v1.12**. The following packages are required:

* `JuMP.jl` (Modeling language for mathematical optimization)
* `SCS.jl` (Splitting Conic Solver for SOCP problems)
* `DataFrames.jl` & `CSV.jl` (Data manipulation)
* `Statistics.jl` & `LinearAlgebra.jl` (Mathematical operations)
* `Plots.jl` (Visualization of efficient frontiers and wealth accumulation)

### Installation
To install the necessary dependencies, open the Julia REPL and run:

```julia
import Pkg
Pkg.add(["JuMP", "SCS", "DataFrames", "CSV", "Plots", "StatsBase"])
