---
layout: post
title: Reading Note of Understanding S4
description: Reading Note of Understanding S4
lang: en
tags: [misc]
published: false
---



Understanding S4
================

State Space Model
-----------------
Goal: model a long sequence.
example: from https://tiktokenizer.vercel.app/
input sequence:
a cat eats rat.<|endoftext|>
tokens:
[64, 8415, 50777, 11494, 13, 100257]
an autoregressive model takes in
[64, 8415, 50777, 11494, 13]
and predicts:
[8415, 50777, 11494, 13, 100257]
Of course these tokens are then get embedded with a giant embedding table to a fixed-length vector with length $l$ where $l=64$ or $l=256$ depending on the size of the language model.

[state space model](https://en.wikipedia.org/wiki/State-space_representation) is an pair of equations, here we show continuous time-invariant case:
$$x'(t)=Ax(t)+Bu(t)$$
$$y(t)=Cx(t)+Du(t)$$
continuous means that $t$ can be any continuous value and time-invariant means that $A,B,C,D$ are the same for all $t$.

S4 uses SSM as a black-box representation in a deep sequence model, where $A,B,C,D$ are parameters learned by gradient descent.

Before we dive into discrete-time SSM and more machine learning related topics, let's first take a closer look to matrices $A$, $B$, and $C$ as they show intuitions why SSM can model a long sequence by learning the relationships between the input $u$, the hidden state $x$, and the output $y$.

**The state matrix $A$** defines the relationship between the current state of the system and its rate of change. In a continuous-time system described by the differential equation $\dot x(t)=Ax(t)+Bu(t)$, where $x(t)$ is the state vector and $u(t)$ is the input vector, the elements of $A$ have the following physical meanings:
 - Diagonal Elements ($a_{ii}$​): These represent the self-influence of state variables on their own rate of change. A negative diagonal element ($a_{ii}<0$) typically indicates a system state that naturally decays or stabilizes over time, whereas a positive diagonal element ($a_{ii}>0$) suggests a state that grows or diverges without external influence. The magnitude of $a_{ii}$​ indicates the rate of decay or growth.

 - Off-diagonal Elements ($a_{ij}$ where $i\neq j$): These elements represent the influence of one state variable on the rate of change of another. For example, $a_{ij}$ where $i\neq j$ indicates how much the $j$-th state affects the rate of change of the $i$-th state. A positive $a_{ij}$ implies that as the $j$-th state increases, the rate of change of the $i$-th state also increases, suggesting a direct or reinforcing relationship. Conversely, a negative $a_{ij}$​ suggests an inverse relationship, where an increase in the $j$-th state causes a decrease in the rate of change of the $i$-th state.

**The input matrix $B$** defines how the input vector $u(t)$ influences the rate of change of the state vector $\dot x(t)$. In the differential equation $\dot x(t)=Ax(t)+Bu(t)$, $B$ modulates the contribution of each input to the state's evolution:

 - Columns of $B$: Each column of $B$ corresponds to an input in the vector $u(t)$ and indicates how that input affects each state variable. If $u(t)$ has multiple components, each component has a column in BB that specifies its influence on the system's states.

 - Magnitude and Sign: The magnitude of the elements in $B$ determines the strength of the influence that each input has on the rate of change of the state variables. A larger absolute value indicates a stronger influence. The sign (positive or negative) indicates whether the input increases (positive) or decreases (negative) the rate of change of the state variables.

 **The output matrix $C$** defines how the state vector $x(t)$ is transformed into the output vector $y(t)$ in the equation $y(t)=Cx(t)+Du(t)$. $C$ determines the relationship between the system's states and its outputs:

 - Rows of $C$: Each row of $C$ corresponds to an output in the vector $y(t)$ and indicates how each state variable contributes to that output. The matrix can filter, combine, or scale state variables to produce the desired outputs.

 - Magnitude and Direction: Similar to $B$, the magnitude of the elements in $C$ indicates the strength of the influence of state variables on each output. A larger absolute value means a stronger influence on the output. The sign shows whether the state variable's contribution to the output is direct (positive) or inverse (negative).

Discrete-time SSM
-----------------
Since the input sequences are discrete tokens, the SSM must be discretized by a step size $\Delta$ that represents the unit resolution of the input. We can view $u_k$ as an implicit sample underlying continuous signal $u(t)$, where $u_k=u(k\Delta)$. For example, if the input sequence is $N=5$ as the example shows, we will choose $\Delta=\frac{1}{N}$, which is 0.2.

To discretize the continuous-time SSM, we use the [bilinear method](https://en.wikipedia.org/wiki/Bilinear_transform), which converts the state matrix $A$ into an approximation $\bar A$. The discrete SSM is hence:
$$\bar A = (I - \Delta/2 \cdot A)^{-1}(I+\Delta/2 \cdot A)$$
$$\bar B = (I - \Delta/2 \cdot A)^{-1}\Delta B$$
$$\bar C = C$$
**Here is the explanation of $\bar A$:**
 - The identity matrix $I$ ensures that the system's state without any changes (i.e., when $A=0$) results in a direct transition from one step to the next without alteration.
 - The term $\Delta/2 \cdot A$ reflects the system's continuous-time dynamics scaled down to fit within the discrete interval defined by $\Delta$. The scaling by $\Delta / 2$ and the subsequent operations effectively "average" the system's behavior over this interval, providing a more accurate approximation than simple direct sampling methods.
 - The multiplication of the two matrices, taking into account the inverse of the first, incorporates both the immediate past and near future into the system's current state estimation in a balanced manner. This balance is crucial for capturing the dynamics accurately, especially in systems where the state changes significantly within each $\Delta$ period.
 
 **Let's also explain the equation of $\bar B$:**
 - Role of $(I - \Delta/2 \cdot A)$: Similar to its role in computing $\bar A$, this term adjusts for the system's dynamics by incorporating the system's behavior considering the immediate past (just before the current timestep). It counterbalances the influence of $A$ to ensure that the effect of input $B$ on the system's state is accurately captured over the discrete interval $\Delta$. By applying the inverse of this matrix, the transformation takes into account the continuous system's dynamics in a way that corrects for the simple direct injection of the inputs at discrete steps.
 - Multiplication by $\Delta$: This operation scales the input matrix $B$ to fit within the discrete time interval defined by $\Delta$. It ensures that the influence of external inputs on the system's state is proportional to the length of the time step, reflecting how input effects accumulate over continuous time in the original system.
 - Overall, $\bar B$: This transformed input matrix allows the discretized system to accurately simulate how inputs $u(t)$ affect the state vector $x(t)$ over each discrete time step, taking into account the continuous dynamics and the discrete interval $\Delta$.

**Finally, the explanation of $\bar C$:**
 - The output matrix $C$ in the continuous-time SSM maps the system's state vector $x(t)$ to the output vector $y(t)$. Because this relationship is direct and does not involve time derivatives (unlike the state equation), the discretization process does not alter $C$.

 We can omit the parameter $D$ for exposition since the term $Du$ can be viewed as a skip connection and is easy to compute.

 This equation is now a *sequence-to-sequence* map $u_k \mapsto y_k$ instead of function-to-function. The state equation is now a recurrence in $x_k$ since now $\bar A$ and $\bar B$ are changes of $A$ and $B$ over the small time step $\Delta$, allowing the discrete SSM to be computed like an RNN. $x_k \in \mathbb{R}^N$ can be viewed as a *hidden state* with transition matrix $\bar A$.
 $$x_k = \bar A x_{k-1} + \bar B u_k$$
 $$y_k = \bar C x_k$$

A Mechanics Example
-------------------
[Mass-Spring-Damper Model](https://en.wikipedia.org/wiki/Mass-spring-damper_model) is a great example to test our current SSM implementation. We are modeling the forward position $y(t)$ of a mass attached to a wall with a spring. Over time, varying force $u(t)$ is applied to this mass. The system is parametrized by mass ($m$), spring constant ($k$), friction constant ($b$). The diffrential equation that ties everything together is:
$$ my''(t) = u(t) - by'(t) - ky(t) $$
It is natural to choose our state $X(t)$ to be a vector contains both $y(t)$ and $y'(t)$. Then we can have the following derivation:
$$ X(t) = \begin{bmatrix} y(t) \\ y'(t) \end{bmatrix} $$
$$ X'(t) = \begin{bmatrix} y'(t) \\ y''(t) \end{bmatrix} $$
$$ X'(t) = \begin{bmatrix} y'(t) \\ \frac{u(t)}{m} - \frac{b}{m}y'(t) - \frac{k}{m}y(t) \end{bmatrix} $$
If we choose our matrix $A$ and $B$ to be:
$$ A = \begin{bmatrix} 0 & 1 \\ -\frac{k}{m} & -\frac{b}{m} \end{bmatrix} $$
$$ B = \begin{bmatrix} 0 \\ \frac{1}{m} \end{bmatrix} $$
We have our first SSM equation:
$$ X'(t) = AX(t) + Bu(t)$$
We can also easily write the second equation by defining $C$:
$$C = \begin{bmatrix} 1 & 0 \end{bmatrix}$$
$$ y(t) = CX(t)$$
