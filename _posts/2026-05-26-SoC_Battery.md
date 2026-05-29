# Battery Model

Base on Coloumb model, State of charge (SOC) of battery is presented in continous space:
$$ 
\frac{dSOC}{dt} = \frac{-I}{Q}
$$
I > 0 means discharge and I < 0 means charging.<br/>
In discrete space, it becomes:
$$ \frac{SOC(k+1)-SOC(k)}{\Delta t} = \frac{-I}{Q} $$
$$ {SOC(k+1)-SOC(k)} = \frac{-I*\Delta t}{Q} $$
$$ {SOC(k+1)} = SOC(k) + \frac{-I*\Delta t}{Q} $$

where I: battery current, Q: battery capacity

We apply Equivalent Circuit Model RC:


$$
\frac{dV_{RC}}{dt}
=
-\frac{V_{RC}}{R_1 C_1}
+
\frac{I}{C_1}
$$

| Symbol | Meaning |
|---|---|
| $R_1$ | polarization resistance |
| $C_1$ | polarization capacitance |
| $V_{RC}$ | RC branch voltage |

---

# Exact ZOH Discretization

Continuous-time state equation:

$$
\dot{x}(t)=Ax(t)+Bu(t)
$$

Under Zero-Order Hold (ZOH), the exact discrete model becomes:

$$
x_{k+1}=A_dx_k+B_du_k
$$

where:

$$
A_d=e^{A\Delta t}
$$

and:

$$
B_d=\int_0^{\Delta t} e^{A\tau}B\,d\tau
$$

---

# Apply To RC Battery Model

Continuous RC equation:

$$
\frac{dV_{RC}}{dt}
=
-\frac{1}{R_1C_1}V_{RC}
+
\frac{I}{C_1}
$$

State-space form:

$$
\dot{x}=Ax+Bu
$$

with:

$$
x = V_{RC}
$$

$$
A = -\frac{1}{R_1C_1}
$$

$$
B = \frac{1}{C_1}
$$

---

# Discrete State Matrix

$$
A_d
=
e^{-\frac{\Delta t}{R_1C_1}}
$$

Define:

$$
a
=
e^{-\frac{\Delta t}{R_1C_1}}
$$

---

# Exact Discrete RC Equation

$$
V_{RC}(k+1)
=
aV_{RC}(k)
+
R_1(1-a)I(k)
$$

# State Vector

$$
x=
\begin{bmatrix}
SOC \\
V_{RC}
\end{bmatrix}
$$

---

# State Function

$$
f(x,u)=
\begin{bmatrix}
f_1(x,u) \\
f_2(x,u)
\end{bmatrix}
$$

where:

$$
f_1
=
SOC_k-\frac{I_k\Delta t}{Q}
$$

$$
f_2
=
aV_{RC,k}+R_1(1-a)I_k
$$

---

# Jacobian F

The state Jacobian is:

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

---

# Step-by-Step Derivatives

## 1. Top-left Element

$$
\frac{\partial f_1}{\partial SOC}
=
\frac{\partial}{\partial SOC}
\left(
SOC_k-\frac{I_k\Delta t}{Q}
\right)
=
1
$$

---

## 2. Top-right Element

$$
\frac{\partial f_1}{\partial V_{RC}}
=
0
$$

because \(f_1\) does not contain \(V_{RC}\).

---

## 3. Bottom-left Element

$$
\frac{\partial f_2}{\partial SOC}
=
0
$$

because \(f_2\) does not contain \(SOC\).

---

## 4. Bottom-right Element

$$
\frac{\partial f_2}{\partial V_{RC}}
=
\frac{\partial}{\partial V_{RC}}
\left(
aV_{RC,k}+R_1(1-a)I_k
\right)
=
a
$$

---

# Final F Matrix

$$
F=
\begin{bmatrix}
1 & 0 \\
0 & a
\end{bmatrix}
$$

where:

$$
a
=
e^{-\frac{\Delta t}{R_1C_1}}
$$

---

# Measurement Function

Battery terminal voltage:

$$
h(x,u)
=
OCV(SOC)-IR_0-V_{RC}
$$

---

# Jacobian H

The measurement Jacobian is:

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

---

# Step-by-Step Derivatives

## 1. Derivative wrt SOC

$$
\frac{\partial h}{\partial SOC}
=
\frac{\partial}{\partial SOC}
\left(
OCV(SOC)-IR_0-V_{RC}
\right)
=
\frac{dOCV}{dSOC}
$$

---

## 2. Derivative wrt VRC

$$
\frac{\partial h}{\partial V_{RC}}
=
\frac{\partial}{\partial V_{RC}}
\left(
OCV(SOC)-IR_0-V_{RC}
\right)
=
-1
$$

---

# Final H Matrix

$$
H=
\begin{bmatrix}
\frac{dOCV}{dSOC} & -1
\end{bmatrix}
$$