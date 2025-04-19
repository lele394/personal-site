> Adapted from a presentation

<style>
.katex-html {
  visibility: hidden;
}
/* Fallback when KaTeX fails */
</style>

# Introduction to Adjoint Control

Adjoint control is a powerful method for solving optimal control problems, particularly when the system dynamics are governed by differential equations. It allows us to compute the gradient of a cost function with respect to a control input efficiently, enabling gradient-based optimization even in high-dimensional systems.

---

## Adjoint Control - Schema

![1](./rsc_adj_ctrl_intro/adj_ctrl_1.drawio.svg)

The core idea is to simulate the **forward dynamics** of a system, then propagate an **adjoint system backward in time** to calculate the gradients necessary for optimization.

---

## Practical Example - Mass-Spring System

### System Overview

We consider a damped mass-spring system with control input:

$$
m \ddot{x} + c \dot{x} + k x = u(t)
$$

Where:
- $m$: mass  
- $c$: damping coefficient  
- $k$: spring stiffness  
- $u(t)$: control force applied to the mass  

This second-order differential equation is converted into **state-space form** for numerical simulation:

$$
\dot{x} = A x + B u
$$

Where:

$$
x = \begin{bmatrix} x_1 \\ x_2 \end{bmatrix} = \begin{bmatrix} r \\ \dot{r} \end{bmatrix}, \quad
A = \begin{bmatrix} 0 & 1 \\ -\frac{k}{m} & -\frac{c}{m} \end{bmatrix}, \quad
B = \begin{bmatrix} 0 \\ \frac{1}{m} \end{bmatrix}
$$

---

## Adjoint Equation

To optimize the control force $u(t)$, we define the **adjoint (costate) system**, which captures how the loss function changes with respect to the system's state:

$$
\dot{\lambda} = -A^T \lambda
$$

with the **terminal condition**:

$$
\lambda_N = x_N - x_{\text{target}}
$$

This means:
- $\lambda_1$: sensitivity of the final loss to the final position
- $\lambda_2$: sensitivity of the final loss to the final velocity

---

## Cost Function

The **loss function** we want to minimize determines how "wrong" your system is compared to the expected state. It's also possible to include other factors of the system. Here, it is composed of two terms:

1. A penalty for deviation from the target state at final time $t_f$
2. A penalty on the total control energy applied over time

$$
J(u) = \textcolor{pink}{\frac{1}{2} * 100 * \|x_N - x_{\text{target}}\|^2} + \textcolor{YellowGreen}{\frac{1}{2} * r * \int_0^{t_f} u(t)^2 \, dt}
$$

Where:
- $x_N$ is the final state at $t = t_f$
- $x_{\text{target}}$ is the desired final state
- The coefficients balance accuracy and control effort (50/50)

---

## Free Evolution (Without Control)

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/no_control.mp4" type="video/mp4">
</video>

Without control, the system evolves freely as expected.

---

## Loss and Error Comparison

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/first_comp.mp4" type="video/mp4">
</video>

**Goal**: drive the system to  
- $r(T_e) = 1.0$  
- $\dot{r}(T_e) = 0.0$  

This comparison shows the difference between the desired and actual outcomes without any control applied.

---

## Adjoint Propagation from the Error

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/adj_back.mp4" type="video/mp4">
</video>

We compute the adjoint by propagating backward in time from the terminal error. This "shadow system" helps us understand how earlier states influenced the final error.

---

## Apply Gradient

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/grad_to_control.mp4" type="video/mp4">
</video>

The gradient $\nabla J_u$ gives us how the control should be adjusted to minimize the loss.

- $\lambda_1$ = sensitivity to position  
- $\lambda_2$ = sensitivity to velocity  

---

## Apply Gradient to Control

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/apply_grad.mp4" type="video/mp4">
</video>

The control is updated using gradient descent:

$$
u(t) \leftarrow u(t) - \alpha \nabla J_u
$$

Where:
- $\alpha$ is the learning rate

---

## Simulate Using Updated Control

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/simulate_with_corr.mp4" type="video/mp4">
</video>

The system is simulated again using the updated control input. You can see the effect of the correction.

---

## New Error With Corrected Control

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/last_comp.mp4" type="video/mp4">
</video>

With the new control applied, the system ends much closer to the desired final state.

---

## Optimization Iterations

Each iteration improves the control force $u(t)$ using the adjoint gradient:

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/it_1.mp4" type="video/mp4">
</video>

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/it_2.mp4" type="video/mp4">
</video>

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/it_5.mp4" type="video/mp4">
</video>

<video width="640" height="360" controls style="display: block; margin: 0 auto;">
  <source src="/assets/data/science/adjoint_control/introduction/rsc_adj_ctrl_intro/it_10.mp4" type="video/mp4">
</video>

We observe:
- Progressive alignment with the target state
- Reduced loss over time
- Efficient convergence using gradient-based updates

---

## Summary

Adjoint control allows for:
- Efficient gradient computation even for large-scale systems
- Gradient-based optimization of controls
- Better handling of systems with complex dynamics

This example with the mass-spring system illustrates how adjoint methods transform control optimization into a tractable and visual process.
