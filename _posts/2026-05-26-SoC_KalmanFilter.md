# Battery EKF (Extended Kalman Filter) for SOC Estimation

---

# 1. State Vector

$$
x_k=
\begin{bmatrix}
SOC_k \\
V_{RC,k}
\end{bmatrix}
$$

---

# 2. Input

Battery current:

$$
u_k = I_k
$$

---

# 3. Measurement

Measured terminal voltage:

$$
y_k = V_{term,k}
$$

---

# 4. State Equations

## SOC Dynamics

$$
SOC_{k+1}
=
SOC_k
-
\frac{I_k \Delta t}{Q}
$$

---

## RC Voltage Dynamics

$$
V_{RC,k+1}
=
aV_{RC,k}
+
R_1(1-a)I_k
$$

where:

$$
a
=
e^{-\frac{\Delta t}{R_1C_1}}
$$

---

# 5. Measurement Equation

$$
V_{term}
=
OCV(SOC)
-
IR_0
-
V_{RC}
$$

---

# 6. Jacobian F

State Jacobian:

$$
F
=
\frac{\partial f}{\partial x}
$$

Expanded form:

$$
F=
\begin{bmatrix}
\frac{\partial f_1}{\partial SOC}
&
\frac{\partial f_1}{\partial V_{RC}}
\\
\\
\frac{\partial f_2}{\partial SOC}
&
\frac{\partial f_2}{\partial V_{RC}}
\end{bmatrix}
$$

Final result:

$$
F=
\begin{bmatrix}
1 & 0 \\
0 & a
\end{bmatrix}
$$

---

# 7. Jacobian H

Measurement Jacobian:

$$
H
=
\frac{\partial h}{\partial x}
$$

Expanded form:

$$
H=
\begin{bmatrix}
\frac{\partial h}{\partial SOC}
&
\frac{\partial h}{\partial V_{RC}}
\end{bmatrix}
$$

Final result:

$$
H=
\begin{bmatrix}
\frac{dOCV}{dSOC}
&
-1
\end{bmatrix}
$$

---

# ==================================
# EKF ALGORITHM
# ==================================

# 8. Prediction Step

## Predict State

$$
\hat x_k^-
=
f(\hat x_{k-1},u_k)
$$

Expanded:

## Predict State

$$
\hat x_k^-
=
\begin{bmatrix}
\widehat{SOC}_k^- \\
\widehat{V}_{RC,k}^-
\end{bmatrix}
=
\begin{bmatrix}
SOC_{k-1}
-
\frac{I_k\Delta t}{Q}
\\
\\
aV_{RC,k-1}
+
R_1(1-a)I_k
\end{bmatrix}
$$

---

## Predict Covariance

$$
P_k^-
=
F_kP_{k-1}F_k^T
+
Q_k
$$

where:

| Symbol | Meaning |
|---|---|
| $P$ | state covariance |
| $Q$ | process noise covariance |

---

# 9. Correction Step

## Predict Terminal Voltage

$$
\hat y_k
=
h(\hat x_k^-,u_k)
$$

Expanded:

$$
\hat V_{term}
=
OCV(\hat SOC_k^-)
-
I_kR_0
-
\hat V_{RC,k}^-
$$

---

## Compute Innovation

Voltage error:

$$
e_k
=
y_k
-
\hat y_k
$$

---

## Innovation Covariance

$$
S_k
=
H_kP_k^-H_k^T
+
R_k
$$

where:

| Symbol | Meaning |
|---|---|
| $R$ | measurement noise covariance |

---

## Kalman Gain

$$
K_k
=
P_k^-H_k^TS_k^{-1}
$$

---

## Correct State

$$
\hat x_k
=
\hat x_k^-
+
K_ke_k
$$

---

## Update Covariance

$$
P_k
=
(I-K_kH_k)P_k^-
$$

---

# Physical Meaning

| Part | Meaning |
|---|---|
| Current integration | short-term SOC tracking |
| Voltage correction | long-term drift correction |
| $F$ | state transition sensitivity |
| $H$ | voltage sensitivity to states |
| $K$ | trust between model and measurement |

---

# Why EKF Works Well For BMS

EKF combines:

- Coulomb counting
- RC battery dynamics
- OCV/SOC relationship
- voltage feedback correction

to estimate SOC robustly even with:
- sensor noise
- model errors
- battery aging