---
layout: single
title: Induction Motor (IM) model for control
tags: [Models, Motor, Control]

header:
    teaser: https://www.engineeringtoolbox.com/docs/documents/832/electric_motor.png

---

### Overview

This is the first of a series of posts about electrical machines modeling. The main goal of these notes is to provide the theoretical foundation for the synthesis of control algorithms.  

### Stator and Rotor voltage

Voltage equations (for $$s=stator, r=rotor$$)

  $$
  \begin{align}  
  v_{ds} &= R_s i_{ds} + \frac{d\psi_{ds}}{dt} - \omega_e \psi_{qs} \\ 
  v_{qs} &= R_s i_{qs} + \frac{d\psi_{qs}}{dt} + \omega_e \psi_{ds} \\
  v_{dr} &= R_s i_{dr} + \frac{d\psi_{dr}}{dt} - \omega_e \psi_{qr} \\ 
  v_{qr} &= R_s i_{qr} + \frac{d\psi_{qr}}{dt} + \omega_e \psi_{dr}
  \end{align}
  $$

- knowing that the flux linkages can be expressed in terms of the currents (for $s=stator, r=rotor, m=mutual$) for the d-axi we get:

  $$
  \begin{align}
  \psi_{ds} &= L_{ls} i_{ds} + L_m(i_{ds}+ i_{dr}) \\
  \psi_{dr} &= L_{lr} i_{dr} + L_m(i_{ds}+ i_{dr}) \\
  \psi_{dm} &= L_m(i_{ds}+ i_{dr})
  \end{align}$$

  and for the q-axis we get:
  
  $$
  \begin{align}
  \psi_{qs} &= L_{ls} i_{qs} + L_m(i_{qs}+ i_{qr}) \\
  \psi_{qr} &= L_{lr} i_{qr} + L_m(i_{qs}+ i_{qr}) \\
  \psi_{qm} &= L_m(i_{qs}+ i_{qr})
  \end{align}
  $$

- substituing $$\psi$$ in the voltage equations we obtain a generic form of the voltages (for $$s=stator, r=rotor$$) :

**TODO - complete correct substitution**

  $$
  \begin{align}  
  v_{ds} &= R_s i_{ds} + L_d \frac{di_d}{dt} - \omega_e L_q i_q \\ 
  v_{qs} &= R_s i_{qs} + L_q \frac{di_q}{dt} - \omega_e L_d i_d + \psi_m \\
  v_{dr} &= R_s i_{ds} + L_d \frac{di_d}{dt} - \omega_e L_q i_q \\ 
  v_{qr} &= R_s i_{qs} + L_q \frac{di_q}{dt} - \omega_e L_d i_d + \psi_m
  \end{align}
  $$

{: .notice--warning}
 **IMPORTANT** -These are the DQ equation widely used?

{: .notice--warning}
 **NODE** -describe single parts of the equations and their meaning

### Rotor torque  

**TODO**  

$$
\begin{align} T &= ψ_{ds}i_{qs} - ψ_{qs} i_{ds}
\end{align}
$$

## Special cases

### Rotor not moving

**TODO**
this is when $$\omega_r=0$$ 

## References

- Bose, 2.2.12.2 - Synchronously Rotating Reference Frame (Kron Equation)
- [Simulink model equations](https://www.mathworks.com/help/sps/ref/inductionmachinesquirrelcage.html)
