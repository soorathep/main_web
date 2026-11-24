# GITT &amp; PITT — what they measure, how to run them, how to calculate

Two intermittent titration techniques for **intercalation / insertion electrodes**. Both step the
cell a little, then wait, across the whole state of charge (SOC), and from the response they give
you the **chemical (apparent) diffusion coefficient** of the working ion in the solid, plus the
**equilibrium (OCV) curve** and the **resistance/overpotential** as a function of SOC.

> Read the caveats at the end first if you only read one part: the number you get is an **apparent**
> diffusion coefficient. Its **absolute value can be wrong by orders of magnitude** because it
> depends on an area and a diffusion length you rarely know well. Use these techniques to compare
> samples **measured the same way**, and report every assumption.

---

## What each one answers

| Question | GITT | PITT |
|---|---|---|
| Chemical diffusion coefficient D vs SOC | ✔ | ✔ |
| Equilibrium potential / OCV vs SOC (thermodynamics) | ✔ (rest voltage) | ✔ (from step charges) |
| Overpotential &amp; internal resistance vs SOC | ✔ | — (indirect) |
| Clues to a change in reaction mechanism | indirect | indirect (current-transient shape) |
| Fine resolution near a phase transition / plateau | coarse | ✔ |
| Time to run | very long (hours–weeks) | long, often faster than GITT |

Both also reveal **kinetic hysteresis** between charge and discharge (run both directions).

---

## GITT — galvanostatic intermittent titration technique

### How to run it (design)
- A series of **constant-current pulses**, each followed by a **rest (relaxation)** to equilibrium.
- Typical conditions (from the literature): current **C/20–C/10** (or ~20 mA g⁻¹), pulse **5–30 min**,
  rest **1–10 h** — long enough that the voltage is flat (dV/dt below a set threshold) at the end.
- Keep the current **small** and the pulse **short** so the step is near-equilibrium and the transient
  voltage is **linear in √t**.
- Run in **both charge and discharge**, over the full voltage window.

### One step, four voltages
`E1` equilibrium before → `E2` after the instant **IR** jump → `E3` end of the pulse →
(IR drop) → `E4` new equilibrium after rest.
- **ΔE_s = E4 − E1** — the steady-state (equilibrium/OCV) change caused by the step.
- **ΔE_τ = E3 − E2** — the transient change during the pulse (i.e. **with the IR drop removed**).

### How to calculate D (Weppner–Huggins)
When the current is small, the pulse short, and E is linear in √t (so `dE/d√t ≈ ΔE_τ/√τ`):

```
        4      ( m_B · V_M )²   ( ΔE_s )²
  D  =  ───  · (───────────)  · (──────)          valid for   τ ≪ L² / D
        π·τ    (  M_B · S  )     ( ΔE_τ )
```

- `τ`  = pulse duration (s)
- `m_B`, `M_B` = mass (g) and molar mass (g mol⁻¹) of the **active material**
- `V_M` = molar volume of the active material (cm³ mol⁻¹)
- `S`  = electrode–electrolyte **contact area** (cm²) — see caveats
- `ΔE_s`, `ΔE_τ` from the step, as defined above

Also get, per step:
- **OCV(SOC)** = `E4` (the rested voltage) — the equilibrium/thermodynamic curve.
- **Overpotential** `η = E_meas − E_eq` (close-circuit minus rested voltage).
- **Internal resistance** `R = η / I_applied`.

### Worked example (do this per step, then plot D vs SOC)
Electrode: `m_B = 1.5 mg`, `M_B = 96 g/mol`, density 4.8 g/cm³ → `V_M = M_B/ρ = 20 cm³/mol`,
`S = 1.13 cm²` (a 12 mm disk), pulse `τ = 600 s`. One step gives `ΔE_s = 8 mV`, `ΔE_τ = 40 mV`.

```
m_B·V_M / (M_B·S) = (1.5e-3 · 20) / (96 · 1.13) = 2.77e-4 cm

D = (4 / (π·600)) · (2.77e-4)² · (8/40)²  ≈  6.5 × 10⁻¹²  cm² s⁻¹
```

Units check: `(cm)² · (dimensionless)² / s = cm²/s`. ✔  Watch the mg→g and the area (say whether S is
geometric or BET — BET can shift D by orders of magnitude).

**Do not do this by hand for 50 steps.** Use `gitt_pitt.py` (in the Resources downloads): it finds
each pulse, extracts E1–E4, computes D vs SOC, and plots it; run `python gitt_pitt.py` for a demo.

---

## PITT — potentiostatic intermittent titration technique

### How to run it (design)
- A **staircase of small potential steps**; hold each step until the current decays to a small
  cut-off, then step again. Integrate the current over a step to get the charge ΔQ (→ incremental
  capacity dQ/dE and the OCV curve).
- Keep steps **small** — `ΔE ≪ RT/F ≈ 25 mV` (commonly 5–20 mV) — so the response is linear and D
  can be read almost continuously vs potential.
- Run charge and discharge over the full window.

### How to calculate D (current transient of each step)
**Long-time diffusion limit:** fit the linear region of `ln|I|` versus time, after faster
modes have decayed and before background current or noise dominates. If the slope is `s`:

```
One-sided planar film (thickness L; blocking back): s = −π²D/(4L²) → D = −4L²s/π²
Homogeneous sphere (radius R; fixed surface):       s = −π²D/R²     → D = −R²s/π²
```

Both limits assume a small step, single-phase diffusion with nearly constant D,
a fixed surface concentration after the step, and negligible charge-transfer and ohmic limitations.
Use the length for the stated geometry. At short times, a diffusion-controlled current may
follow `|I| ∝ t^(−1/2)` (Cottrell); obtaining D from that slope also requires a
geometry-appropriate relation between current, incremental charge and diffusion length.

**Script limitation (`gitt_pitt.py`):** The current script uses only the one-sided planar-film prefactor for PITT. Do not use a particle radius as its input length. Its synthetic demo has a prescribed dip in D.

### What the transient can tell you
The transient shape or a dip in apparent D can suggest a change worth investigating, but neither identifies a phase transition without independent evidence.
Interfacial kinetics, ohmic drop or geometry can also distort the apparent D.

---

## What to report (both)

- [ ] Technique, **pulse/step size**, **pulse/step duration**, **rest criterion** (the dV/dt or
  current cut-off you used), **temperature**, and direction (charge/discharge).
- [ ] Every quantity in the equation, including **m_B, M_B, V_M, S** for GITT or the **geometry and length (L or R)** for PITT, and how the area or length was determined.
- [ ] A **representative single step** (E–t, and E–√t for GITT / ln|I|–t for PITT) to justify the
  linear-fit assumption.
- [ ] **D vs SOC** for charge **and** discharge (log scale), not a single headline number.
- [ ] Call D an **apparent / chemical** diffusion coefficient, and state the range, not one value.
- [ ] Follow the general **battery reporting checklist** (loading, n cells, etc.) as well.

---

## Caveats — read these

- **The absolute D is not trustworthy.** A porous composite electrode has a particle-size
  distribution and an unknown true active area `S`; using geometric vs BET area shifts D by **orders
  of magnitude**. The value is meaningful mainly for **relative** comparison of samples run under
  identical conditions.
- **Two-phase and conversion intervals violate the single-phase assumptions.** Do not interpret
  a simple GITT or PITT fit there as a single-phase diffusion coefficient.
- **`τ ≪ L²/D` must actually hold**; if the pulse is too long the √t linearity fails.
- **Equilibrium must really be reached** during rest, or ΔE_s (and the OCV) is wrong. Conversion
  electrodes can need days.
- Before comparing GITT and PITT values, check the geometry, fit window and other model assumptions.

---

## References to read (start with the first two)

1. **J. Kim, S. Park, S. Hwang, W.-S. Yoon, "Principles and Applications of the Galvanostatic
   Intermittent Titration Technique for Lithium-ion Batteries," *J. Electrochem. Sci. Technol.* 13,
   19–31 (2022).** Open access, beginner-friendly, full derivation. https://doi.org/10.33961/jecst.2021.00836
2. **W. Weppner &amp; R. A. Huggins, *J. Electrochem. Soc.* 124, 1569 (1977).** The original GITT paper.
3. Y. Zhu &amp; C. Wang, "Strain accommodation and potential hysteresis of LiFePO₄…" and related
   PITT analyses — for the two-phase transient shape.
4. **C. Montella, "Apparent diffusion coefficient of intercalated species measured with
   PITT: A simple formulation," *Electrochim. Acta* 51, 3102–3111 (2006).**
   https://doi.org/10.1016/j.electacta.2005.08.046. Discusses departures from ideal diffusion conditions.
5. **A. J. Bard &amp; L. R. Faulkner, *Electrochemical Methods*, 2nd/3rd ed.** — Cottrell equation and
   diffusion fundamentals behind both techniques.
6. **Cautionary reading:** papers titled *"Spurious chemical diffusion coefficients of Li⁺… GITT"*
   (Electrochim. Acta, 2004) and *"Spurious potential dependence… PITT"* (Electrochim. Acta, 2002) —
   why the absolute numbers can be wrong.
7. K. J. Griffith, C. P. Grey et al., *Nature* 559, 556 (2018) — a clean example of measuring and
   interpreting fast solid-state diffusion.
8. **BioLogic Application Note 70** — practical setup of EIS / PITT / GITT on a potentiostat.
