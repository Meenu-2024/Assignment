# 📘 Parametric Curve Fitting 

## 🎯 Problem Statement

Estimate the unknown parameters **θ**, **M**, and **X** in the following **parametric equations** of a curve:

\[
x(t;\theta,M,X) = t\cos(\theta) - e^{M|t|}\sin(0.3t)\sin(\theta) + X
\]

\[
y(t;\theta,M,X) = 42 + t\sin(\theta) + e^{M|t|}\sin(0.3t)\cos(\theta)
\]

### Unknowns and Ranges

| Parameter | Symbol | Range |
|------------|---------|-------|
| Theta | θ | 0° < θ < 50° |
| M | M | -0.05 < M < 0.05 |
| X | X | 0 < X < 100 |

Parameter **t** varies in the range:
\[
6 < t < 60
\]

The objective is to minimize the **mean L1 distance** between observed and predicted curve points:

\[
\text{Mean L1} = \frac{1}{N}\sum_{i=1}^{N} (|x_i - \hat{x}_i| + |y_i - \hat{y}_i|)
\]

---

## 📂 Input Data

The dataset **`xy_data.csv`** contains points sampled from the curve within \(6 < t < 60\).

The code supports multiple formats:
- Columns: `t, x, y`
- Columns: `x, y` (no `t` column)
- Unnamed numeric columns

If the **t** column is missing, the program constructs it uniformly as:
\[
t_i = 6 + \frac{(60-6)(i-1)}{N-1}, \quad i = 1, 2, …, N
\]

All data are filtered to retain only points within \(6 < t < 60\).

---

## ⚙️ Model Used

\[
\begin{aligned}
x_{\text{pred}} &= t\cos(\theta) - e^{M|t|}\sin(0.3t)\sin(\theta) + X,\\
y_{\text{pred}} &= 42 + t\sin(\theta) + e^{M|t|}\sin(0.3t)\cos(\theta)
\end{aligned}
\]

The variable **θ** is supplied in degrees and converted to radians internally during computation.

---

## 🧮 Optimization Objective

The model minimizes the **mean L1 loss**:

\[
L(\theta, M, X) = \frac{1}{N}\sum_{i=1}^{N}(|x_i - x_{\text{pred},i}| + |y_i - y_{\text{pred},i}|)
\]

### Why L1?
- L1 (mean absolute error) is **robust to outliers**.
- It directly aligns with the assignment’s **evaluation metric**.

---

## 🚀 Optimization Strategy

The optimization process uses **two stages** for accuracy and robustness.

### 1️⃣ Global Optimization — Differential Evolution (DE)
- Explores the full parameter space to avoid local minima.
- Parameters:
  - `popsize = 30`
  - `maxiter = 1800`
  - `tol = 1e-8`
- Uses the `"best1bin"` strategy from SciPy.
- Produces a good approximate global solution.

### 2️⃣ Local Refinement — L-BFGS-B
- Starts from the DE result to refine and fine-tune parameters.
- Parameters:
  - `ftol = 1e-12`
  - `maxiter = 20000`
- Achieves a precise local minimum.

This combination ensures both **exploration (DE)** and **fine convergence (L-BFGS-B)**.

---

## 🧩 Complete Process and Steps Followed

### Step 1: Data Loading
- Load `xy_data.csv` using **Pandas**.
- Automatically detects column headers.
- If `t` is missing, creates uniform `t = np.linspace(6, 60, N)`.
- Filters all rows to ensure \(6 < t < 60\).

### Step 2: Model Function Definition
Implements the given mathematical equations for \(x(t)\) and \(y(t)\) using vectorized **NumPy** operations for efficient computation.

### Step 3: Loss Function Definition
Defines the **mean L1 distance** as the objective function to minimize.  
This is robust to data irregularities and matches the evaluation criterion.

### Step 4: Global Optimization (Differential Evolution)
Runs a **population-based search** across the parameter space to find a strong global candidate solution.

### Step 5: Local Optimization (L-BFGS-B)
Uses the global DE result as the initial guess, refining it with a **gradient-based method** for precise convergence.

### Step 6: Result Computation
After optimization:
- Compute predicted `(x_pred, y_pred)` for each `t`.
- Calculate and print the **mean L1** loss value.
- Save all results and diagnostics.

### Step 7: Visualization and Output Saving
The program generates:
- `fitted_points.csv` — comparison of experimental vs predicted values.
- `fit_overlay.png` — overlay plot of data points and fitted curve.
- `fit_summary.json` — JSON summary of best-fit parameters and mean L1.
- `desmos_output.txt` — Desmos/LaTeX formatted submission string.

---

## 🧾 Desmos Output Format

The final output for submission is automatically written as:
(tcos(θ_rad) - exp(Mabs(t))sin(0.3t)sin(θ_rad) + X,
42 + tsin(θ_rad) + exp(Mabs(t))*sin(0.3t)*cos(θ_rad))

---

## 📊 Output Files Generated

All results are automatically saved in the folder **`fit_outputs_colab/`**.

| File Name | Description |
|------------|-------------|
| `fitted_points.csv` | Actual vs predicted coordinates |
| `fit_overlay.png` | Overlay of fitted curve with data points |
| `fit_summary.json` | Optimized parameters and mean L1 value |
| `desmos_output.txt` | Desmos/LaTeX-ready parametric curve string |

---

## 💻 How to Run in Google Colab

1. Open [Google Colab](https://colab.research.google.com/).
2. Copy and paste the provided Python script into a Colab cell.
3. Run the cell.
4. When prompted, upload your `xy_data.csv` file.
5. Wait for optimization to complete (takes 2–5 minutes).
6. Download results from the `fit_outputs_colab/` folder.

---

## 🔍 Diagnostics & Validation

After fitting, residuals are computed for each data point:

\[
r_i = |x_i - \hat{x}_i| + |y_i - \hat{y}_i|
\]

### How to validate:
- Check the **mean** and **median** residuals — smaller values imply better fit.
- Review the **maximum residual** to detect any outliers.
- Inspect `fit_overlay.png` — the predicted curve should closely match the data.

---

## ⚙️ Reproducibility and Tuning

- The random seed is fixed (`seed=0`) for reproducibility.
- You can tune:
  - `de_pop` (population size)
  - `de_it` (number of iterations)
- Increasing these improves accuracy but increases runtime.
- Results remain deterministic for the same random seed.

---

## 🧠 Example Final Results

| Parameter | Symbol | Example Value |
|------------|---------|---------------|
| Theta | θ | 28.11877332° |
| M | M | 0.02138680 |
| X | X | 54.89905081 |
| Mean L1 | — | 25.20758073 |

*(Values may vary slightly based on dataset and optimization seed.)*

---

## 🏁 Conclusion

This implementation accurately estimates the unknown parameters **(θ, M, X)** of the given parametric curve by minimizing the **mean L1 distance** between predicted and observed points.

### ✅ Highlights
- Robust to missing `t` column.
- Combines **global (DE)** and **local (L-BFGS-B)** optimization.
- Automatically generates **Desmos-ready output**.
- Produces interpretable residual diagnostics and visual overlays.

---


