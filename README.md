# Kerr Spacetime & Radiative Accretion Workstation

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![WebGL 2.0](https://img.shields.io/badge/WebGL-2.0-green.svg)](https://www.khronos.org/webgl/)
[![Three.js r160](https://img.shields.io/badge/Three.js-r160-black.svg)](https://threejs.org/)
[![Physics: General Relativity](https://img.shields.io/badge/Physics-General%20Relativity-informational.svg)]()

An interactive, single-file numerical relativity and astrophysical radiative transfer workstation built with **WebGL 2.0 (Three.js r160)** and custom **GLSL shaders**. 

Designed for researchers, physicists, and graduate students, this simulator evaluates the exact **Boyer-Lindquist metric tensor** for an axisymmetric rotating black hole (Kerr spacetime), integrates timelike and null geodesics via a 4th-order Runge-Kutta (RK4) solver, and models relativistic accretion disk emission using the **Novikov-Thorne** thin-disk formalism.

---

## Key Features & Physical Formalism

### 1. Exact Kerr Metric Tensor Evaluation ($G = c = 1$)
The workstation computes the covariant metric tensor components $g_{\mu\nu}(r, \theta)$ in Boyer-Lindquist coordinates $(t, r, \theta, \phi)$ in real time:

$$ds^2 = -\left(1 - \frac{2Mr}{\Sigma}\right)dt^2 - \frac{4Mar\sin^2\theta}{\Sigma}dt\,d\phi + \frac{\Sigma}{\Delta}dr^2 + \Sigma\,d\theta^2 + \left(r^2 + a^2 + \frac{2Ma^2r\sin^2\theta}{\Sigma}\right)\sin^2\theta\,d\phi^2$$

where:
* $\Sigma \equiv r^2 + a^2\cos^2\theta$
* $\Delta \equiv r^2 - 2Mr + a^2$
* $a \equiv J/M \in [0, M)$ is the specific Kerr spin parameter ($a_* = a/M$).

### 2. Characteristic Relativistic Horizons & Critical Radii
Following the analytical derivations of **Bardeen, Press, & Teukolsky (1972)**, the engine dynamically solves and renders:
* **Outer Event Horizon ($r_+$) & Inner Cauchy Horizon ($r_-$):**
  $$r_\pm = M \pm \sqrt{M^2 - a^2}$$
* **Static Limit / Ergosphere Boundary ($r_{\text{ergo}}$):**
  $$r_{\text{ergo}}(\theta) = M + \sqrt{M^2 - a^2\cos^2\theta}$$
* **Prograde & Retrograde Photon Sphere Orbits ($r_{\text{ph}}^\pm$):**
  $$r_{\text{ph}}^\pm = 2M \left\{ 1 + \cos\left[ \frac{2}{3}\arccos\left(\mp a_*\right) \right] \right\}$$
* **Innermost Stable Circular Orbit ($r_{\text{ISCO}}^\pm$):**
  $$r_{\text{ISCO}} = M \left( 3 + Z_2 \mp \sqrt{(3 - Z_1)(3 + Z_1 + 2Z_2)} \right)$$
  with $Z_1 = 1 + (1 - a_*^2)^{1/3}\left[(1 + a_*)^{1/3} + (1 - a_*)^{1/3}\right]$ and $Z_2 = \sqrt{3a_*^2 + Z_1^2}$.

### 3. Novikov-Thorne Thin Accretion Disk & Liouville Invariant
The accretion disk shader implements the relativistic thin-disk temperature and flux profile $F(r)$ vanishing at the boundary $r = r_{\text{ISCO}}$. Observed specific intensity $I_{\text{obs}}$ is computed via Liouville's theorem along null rays:

$$I_{\text{obs}}(\nu_{\text{obs}}) = g^4 I_{\text{em}}(\nu_{\text{em}}), \qquad g = \frac{\nu_{\text{obs}}}{\nu_{\text{em}}} = \frac{1}{u^t (1 - \Omega_K b)}$$

where $\Omega_K = \frac{\sqrt{M}}{r^{3/2} \pm a\sqrt{M}}$ is the Keplerian angular velocity, $b = L_z / E$ is the photon impact parameter, and $g$ is the total relativistic Doppler + gravitational redshift factor.

### 4. RK4 Timelike Geodesic Integrator & Phase-Space Oscilloscope
Test particle trajectories are integrated in the equatorial plane ($\theta = \pi/2$) using conserved specific energy $E = -u_t$ and specific azimuthal angular momentum $L_z = u_\phi$:

$$\frac{1}{2}\left(\frac{dr}{d\tau}\right)^2 + V_{\text{eff}}(r; a, E, L_z) = \frac{E^2 - 1}{2}$$

$$V_{\text{eff}}(r) = -\frac{M}{r} + \frac{L_z^2 - a^2(E^2 - 1)}{2r^2} - \frac{M(L_z - aE)^2}{r^3}$$

Supports bound eccentric orbits, apsidal relativistic precession, **Zoom-Whirl** separatrix orbits near the unstable circular orbit ($r_{\text{ibco}}$), and plunging trajectories across $r_+$.

### 5. Thermodynamic, Curvature & Electrodynamic Invariants
Real-time computation of CGS/SI and geometric invariants:
* **Kretschmann Curvature Scalar ($K = R_{\alpha\beta\gamma\delta}R^{\alpha\beta\gamma\delta}$):**
  $$K(r, \theta) = \frac{48M^2}{\Sigma^6}\left(r^6 - 15a^2r^4\cos^2\theta + 15a^4r^2\cos^4\theta - a^6\cos^6\theta\right)$$
* **Horizon Angular Velocity (Frame Dragging):** $\Omega_H = \frac{a}{2Mr_+}$
* **Surface Gravity ($\kappa$), Hawking Temperature ($T_H$), & Bekenstein-Hawking Entropy ($S_{\text{BH}}/k_B$).**
* **Blandford-Znajek Relativistic Jet Luminosity ($P_{\text{BZ}}$):** Scaled by horizon magnetic flux $\Phi_{\text{BH}}$ and $\Omega_H^2$.

---

## Radiative Transfer Colormaps (GLSL)

The top command bar allows switching between four physically calibrated spectral transfer representations:
1. **`0: EHT 230 GHz Synchrotron (afmhot)`**: Perceptually uniform black-red-orange-yellow-white colormap matching Event Horizon Telescope VLBI reconstructions of M87* and Sgr A*.
2. **`1: Planckian Blackbody Locus (3000K–18000K)`**: True chromaticity mapping across the thermal emission profile of the inner accretion disk.
3. **`2: Bolometric Specific Intensity (I_ν Monochrome)`**: Pure high-contrast luminance representation of bolometric flux $F(r) \propto g^4 T_{\text{eff}}^4$.
4. **`3: Relativistic Redshift Factor g (Diverging)`**: Isolates the kinematic Doppler beaming vs. gravitational redshift across the disk plane (blueshifted approaching limb vs. redshifted receding limb).

---

## Interface & Controls Guide

### Left Dock: Spacetime & Orbital Parameters
* **Kerr Spin Parameter ($a_* = J/M^2$):** Adjusts black hole rotation from Schwarzschild ($a_* = 0.000$) to near-extremal Kerr ($a_* = 0.998$, Thorne limit).
* **Primary Mass ($M / M_\odot$):** Log-scale mass selector from stellar-mass black holes to supermassive regimes ($6.5 \times 10^9 M_\odot$, M87* scale) for SI/CGS conversion.
* **Eddington Accretion Rate ($\dot{m} = \dot{M}/\dot{M}_{\text{Edd}}$):** Controls disk radiative flux and Blandford-Znajek jet power.
* **Observer Inclination ($i_{\text{obs}}$):** Sets polar viewing angle ($0^\circ$ face-on to $89^\circ$ edge-on).
* **Geodesic Initial Conditions ($r_0/M$, $L_z/M$, $e_{\text{rel}}$):** Configures the test particle's initial radial coordinate, angular momentum, and eccentricity.
* **Orbital Presets:**
  * `ISCO Circular`: Places the test particle exactly on the marginally stable circular orbit $r_{\text{ISCO}}$.
  * `Zoom-Whirl`: Configures a high-eccentricity separatrix orbit exhibiting extreme periastron whirl around the photon sphere before zooming out.
  * `Plunge Orbit`: Sets $L_z < L_{\text{crit}}$ to demonstrate direct capture across the outer event horizon $r_+$.

### Top Command Bar: Manifold Decomposition & Diagnostics
* **Manifold Layer Toggles:** Independently toggle the Event Horizon ($r_+$), Ergosphere ($r_{\text{ergo}}$), Photon Sphere ($r_{\text{ph}}$), Accretion Disk, Blandford-Znajek Jets, Flamm's Embedding Paraboloid, and 3D Coordinate Annotations.
* **`Decompose Manifold` (GSAP):** Separates the concentric spacetime boundaries vertically along the spin axis for topological inspection of the inner/outer horizons, ergosphere, and embedding diagram.
* **`Export State JSON`:** Copies the complete current spacetime metric tensor, horizon radii, thermodynamic quantities, and instantaneous geodesic state vector to the clipboard in structured JSON format.

### Right Dock: Tensor & Phase-Space Diagnostics
* **Characteristic Radii & Invariants Table:** Live numerical readout in geometric units ($r/M$) and physical units.
* **Covariant Metric Matrix $g_{\mu\nu}(r, \theta)$:** Live $4 \times 4$ matrix evaluated at the instantaneous coordinates of the test particle.
* **Dual Oscilloscope Canvas:**
  * *Top Subplot:* Effective potential curve $V_{\text{eff}}(r/M)$ with live energy level $(E^2-1)/2$ and particle position marker.
  * *Bottom Subplot:* Symplectic radial phase portrait $(r/M, dr/d\tau)$.

---

## Getting Started (Zero Dependencies / Local Execution)

The entire workstation is self-contained in a single `index.html` file using ES6 Module Import Maps.

### Option 1: Direct Browser Execution
Simply open `index.html` in any modern Chromium, Firefox, or WebKit browser supporting WebGL 2.0.

### Option 2: Local Development Server (Recommended)
```bash
# Clone the repository
git clone [https://github.com/YOUR_USERNAME/kerr-spacetime-workstation.git](https://github.com/YOUR_USERNAME/kerr-spacetime-workstation.git)
cd kerr-spacetime-workstation

# Run with Python 3
python3 -m http.server 8080

# Or using Node.js (npx)
npx serve .
```
Then navigate to `http://localhost:8080` in your browser.

---

## Literature & Mathematical References

1. **Boyer, R. H., & Lindquist, R. W. (1967).** *Maximal Analytic Extension of the Kerr Metric.* Journal of Mathematical Physics, 8(2), 265–281.
2. **Bardeen, J. M., Press, W. H., & Teukolsky, S. A. (1972).** *Rotating Black Holes: Locally Nonrotating Frames, Energy Extraction, and Scalar Synchrotron Radiation.* The Astrophysical Journal, 178, 347–370.
3. **Novikov, I. D., & Thorne, K. S. (1973).** *Astrophysics of Black Holes.* In *Black Holes (Les Astres Occlus)*, pp. 343–450.
4. **Page, D. N., & Thorne, K. S. (1974).** *Disk-Accretion onto a Black Hole. Time-Averaged Structure of Accretion Disk.* The Astrophysical Journal, 191, 499–506.
5. **Blandford, R. D., & Znajek, R. L. (1977).** *Electromagnetic extraction of energy from Kerr black holes.* Monthly Notices of the Royal Astronomical Society, 179(3), 433–456.
6. **Event Horizon Telescope Collaboration (2019).** *First M87 Event Horizon Telescope Results. I. The Shadow of the Supermassive Black Hole.* The Astrophysical Journal Letters, 875(1), L1.

---

## License

Distributed under the **MIT License**. See `LICENSE` for more information.