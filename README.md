# w — engineering sims in the browser

Two self-contained simulators. No build step, no dependencies: open the HTML file.

| Project | Path | What it is |
|---|---|---|
| Tailing Box Twin | [`index.html`](index.html) | PID vs MPC level control on twin flotation tailing boxes |
| Lattice Wind Tunnel | [`wind-tunnel/index.html`](wind-tunnel/index.html) | 2D CFD: D2Q9 lattice Boltzmann solver with LES, force balance and vortex-shedding analysis |

---

# Tailing Box Twin

A live, in-browser twin of two identical flotation tailing boxes fed by the same tailings stream.
Box A is held by a **PID** level loop, Box B by a **model predictive controller (MPC)**.
Hit the feed with a surge or a loss and watch which one keeps level and which one keeps the pump calm.

**Run it:** open `index.html` in any browser. No build, no dependencies.

## What's inside

- Animated tank: slurry inflow, surface waves, rotating VSD pump impeller, HH/LL alarm limits
- Upsets: feed surge (+35%), feed loss (−40%), setpoint step, natural feed variation
- Live tuning: PID (Kc, Ti, Td, optional feed-forward) and MPC (prediction horizon, move suppression λ)
- 15-minute trend of level, pump speed and flow, plus a scorecard (IAE, worst deviation, pump travel, discharge σ, time in alarm)

## Process model

| Parameter | Value |
|---|---|
| Box area | 25 m² |
| Box height | 5 m (HH 4.3 m, LL 0.7 m) |
| Nominal feed | 3,200 m³/h |
| Pump capacity | 4,800 m³/h at 100% VSD |
| VSD ramp limit | 1.5 %/s |
| Control interval | 2 s |

Level: `dh/dt = (Q_in − Q_out) / A`, with pump flow depending on suction head
(`Q_out = Q_max · u · (0.82 + 0.072·h)`). The MPC's internal model ignores the suction-head term,
so there is real plant–model mismatch, handled by a bias (disturbance) estimator.

The MPC solves an unconstrained least-squares problem over a 4-move control horizon every 2 s,
then clips the move to ±6 % and the output to 15–100 %. It uses the feed flow meter (FIT-3100) as a measured disturbance.

## Host on GitHub Pages

Settings → Pages → Deploy from branch → `main` / root. The sim will be live at
`https://<user>.github.io/<repo>/`.

---

# Lattice Wind Tunnel

A real 2D CFD solver written from scratch in plain JavaScript. It runs a **D2Q9 lattice Boltzmann method** on a 360 × 144 grid (466,560 distribution functions) and renders vorticity, speed or pressure live, with tracer particles.

## Numerics

- **Collision:** BGK single-relaxation-time with a **Smagorinsky LES** eddy-viscosity model. The local relaxation time comes from the non-equilibrium stress tensor:
  `τ_eff = ½ (τ₀ + √(τ₀² + 18 C_s² Q / ρ))`, where `Q = √(2 Π:Π)` and `C_s = 0.1`. This keeps the solver stable up to Re ≈ 2000 on a coarse grid.
- **Streaming:** pull scheme with **half-way bounce-back** (no-slip) on every solid link.
- **Boundaries:** equilibrium velocity inlet with a soft start ramp, equilibrium far-field top and bottom walls, zero-gradient outlet.
- **Forces:** **momentum-exchange method**, summed over all fluid–solid links each step. `F = Σ 2 f_ō(x) e_ō`.
- **Coefficients:** `C_d = F_x / (½ ρ U² L)` and `C_l = F_y / (½ ρ U² L)`, with L the cylinder diameter, airfoil chord or plate height.
- **Strouhal number:** taken from the mean period between upward zero crossings of the de-meaned lift signal, `St = f L / U`.
- **Viscosity from Re:** `ν = U L / Re`, `τ₀ = 3ν + ½`, inlet `U = 0.1` lattice units (Ma ≈ 0.17).

## Validation (cylinder, Re = 200, 17% blockage)

| Quantity | This solver | Unconfined reference |
|---|---|---|
| Mean C_d | ≈ 1.70 | ≈ 1.34 |
| C_l amplitude | ≈ 0.83 | ≈ 0.7 |
| Strouhal | ≈ 0.210 | ≈ 0.197 |

Blockage from the tunnel walls accelerates the flow around the body, so the confined values sit above the free-stream references. That trend matches what the literature reports.

## Things to try

- Sweep Re from 30 to 60 on the cylinder and watch the wake turn unstable near Re ≈ 47 (Hopf bifurcation).
- Put the NACA 0012 at 15–20° and watch it stall: lift drops and the wake starts shedding.
- Use **Tandem** to see the downstream cylinder sit in the upstream one's wake (drag shielding).
- Pick **Draw wall** and sketch your own shape. Re and the coefficients rescale with its projected height.
