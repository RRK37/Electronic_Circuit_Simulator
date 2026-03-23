# Electronic Circuit Simulator

Filters, amplifiers, transmission lines, and impedance matching networks all exhibit behaviour that only reveals itself across frequency. This simulator models arbitrary linear networks as cascaded **ABCD transmission matrices** -- a formulation where resistors, capacitors, inductors, and conductances all reduce to the same 2x2 matrix framework, and complex multi-stage circuits assemble by matrix multiplication alone. A single RC low-pass and a 100-section LC ladder network (a discrete transmission line) require no different treatment -- just more multiplications.

Reads SPICE-like netlists, sweeps across frequency, and produces full impedance, voltage, current, power, and gain characteristics as CSV output. Built in Python with NumPy.

---

## The Mathematics

### Two-Port Network Representation

Every linear circuit element is represented as a 2x2 transmission (ABCD) matrix relating input and output port voltages and currents:

$$
\begin{bmatrix} V_1 \\ I_1 \end{bmatrix} = \begin{bmatrix} A & B \\ C & D \end{bmatrix} \begin{bmatrix} V_2 \\ I_2 \end{bmatrix}
$$

A **series impedance** Z and a **shunt admittance** Y = 1/Z yield:

$$
\mathbf{T}_{\text{series}} = \begin{bmatrix} 1 & Z \\ 0 & 1 \end{bmatrix}, \qquad \mathbf{T}_{\text{shunt}} = \begin{bmatrix} 1 & 0 \\ Y & 1 \end{bmatrix}
$$

### Frequency-Dependent Impedance

Reactive components introduce complex, frequency-dependent impedances where $\omega = 2\pi f$:

$$
Z_R = R, \qquad Z_C(\omega) = \frac{1}{j\omega C} = \frac{-j}{\omega C}, \qquad Z_L(\omega) = j\omega L
$$

At each frequency point, every component's ABCD matrix is recomputed with its current impedance value -- meaning the entire cascade is frequency-aware.

### Cascading Networks

The power of the ABCD formulation is that cascaded stages combine by simple matrix multiplication:

$$
\mathbf{T}_{\text{total}}(f) = \prod_{k=1}^{N} \mathbf{T}_k(f) = \mathbf{T}_1(f) \cdot \mathbf{T}_2(f) \cdots \mathbf{T}_N(f)
$$

This scales cleanly -- the simulator handles everything from a single RC filter to a 100-section LC ladder network (a discrete transmission line model) with the same algorithm.

### Derived Circuit Parameters

From the cascaded ABCD parameters and the source/load terminations ($R_s$, $R_L$, $V_s$), all circuit quantities follow:

$$
Z_{\text{in}} = \frac{A \cdot R_L + B}{C \cdot R_L + D}, \qquad Z_{\text{out}} = \frac{D \cdot R_s + B}{C \cdot R_s + A}
$$

$$
V_{\text{in}} = \frac{Z_{\text{in}}}{Z_{\text{in}} + R_s} \cdot V_s, \qquad V_{\text{out}} = \frac{V_{\text{in}}}{A + B / R_L}
$$

$$
A_v = \frac{V_{\text{out}}}{V_{\text{in}}} = \frac{R_L}{A \cdot R_L + B}, \qquad A_i = \frac{I_{\text{out}}}{I_{\text{in}}} = \frac{1}{C \cdot R_L + D}
$$

Complex power at each port uses the conjugate product:

$$
S = V \cdot I^* = P + jQ
$$

where $P$ is real (dissipated) power and $Q$ is reactive power.

---

## Usage

```
python mainScript.py <netlist.net>
```

### Netlist Format

```
<CIRCUIT>
n1=1 n2=2 R=8.55k
n1=2 n2=0 C=3.18n
</CIRCUIT>

<TERMS>
VT=5 RS=50 RL=75
Fstart=10 Fend=10e6 Nfreqs=50
</TERMS>

<OUTPUT>
Vin V
Vout dBV
Zin Ohms
Av dB
</OUTPUT>
```

- **`<CIRCUIT>`** -- Component definitions with node connections. Series elements connect between two non-ground nodes; shunt elements connect to ground (node 0).
- **`<TERMS>`** -- Source (Thevenin `VT`/`RS` or Norton `IN`/`GS`), load `RL`, and frequency sweep (linear or logarithmic).
- **`<OUTPUT>`** -- Requested parameters with units. Supports `V`, `A`, `Ohms`, `W`, `dB`, `dBV`, `dBA`, `dBW` and SI prefixes.

### Output

CSV files with real/imaginary (or magnitude-dB/phase) columns for each requested parameter across the frequency sweep.

---

## Supported Parameters

| Symbol | Description | Formula |
|--------|------------|---------|
| `Vin`  | Input voltage | $Z_{\text{in}} \cdot V_s / (Z_{\text{in}} + R_s)$ |
| `Vout` | Output voltage | $V_{\text{in}} / (A + B/R_L)$ |
| `Iin`  | Input current | $V_s / (Z_{\text{in}} + R_s)$ |
| `Iout` | Output current | $V_{\text{out}} / R_L$ |
| `Pin`  | Input power | $V_{\text{in}} \cdot I_{\text{in}}^*$ |
| `Pout` | Output power | $V_{\text{out}} \cdot I_{\text{out}}^*$ |
| `Zin`  | Input impedance | $(AR_L + B)/(CR_L + D)$ |
| `Zout` | Output impedance | $(DR_s + B)/(CR_s + A)$ |
| `Av`   | Voltage gain | $R_L / (AR_L + B)$ |
| `Ai`   | Current gain | $1 / (CR_L + D)$ |

---

## Dependencies

- Python 3
- NumPy
- Matplotlib (for plotting)
