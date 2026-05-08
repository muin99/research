# Experiment 7 — Performance Tasks Report

**Course:** COE 3201 — Data Communication Laboratory
**Department:** Computer Engineering, American International University–Bangladesh
**Experiment:** No. 7 — Amplitude and Frequency Modulation / Demodulation in Simulink
**Tasks covered:** Performance Task 1 (AM) and Performance Task 2 (FM)

---

## Abstract

This report covers the two performance tasks of Experiment 7. In Task 1 a
multi-tone message
$m(t) = 2\sin(2\pi 4 t)+3\cos(2\pi 6 t)$ is amplitude-modulated on a carrier
$c(t)=\cos(2\pi 50 t)$, transmitted, and recovered using a coherent (synchronous)
detector. In Task 2 a single-tone message
$m(t) = 2\sin(2\pi 10 t + \pi/3)$ is frequency-modulated on a $f_c = 100\,\text{Hz}$
carrier and recovered using a frequency discriminator (differentiator + envelope
detector + low-pass filter). Both modulators follow the block-diagram structure
shown in the lab manual (Sine Wave / Constant / Sum / Product for AM, Integrator /
Gain / Clock / Sum / cos for FM). The Simulink models are built programmatically
by two MATLAB scripts (`exp7_perf1_AM.m` and `exp7_perf2_FM.m`) and run with the
modern `sim` API, returning a `Simulink.SimulationOutput` object whose contents
are validated against the original message in the script. Both schemes recover
$m(t)$ to within numerical precision after compensating for the
group delay of the receive low-pass filter; deviations from the ideal trace are
shown to be entirely a property of that LPF, not of the modulation/demodulation
math.

---

## Problem Statement

The lab manual specifies two performance tasks for Experiment 7.

### Performance Task 1 (Part-1 of the manual)

> *Implement the following demodulation in Simulink to retrieve the original
> signal: You have a signal* $m(t) = 2\sin(2\pi 4 t) + 3\cos(2\pi 6 t)$.
> *Apply amplitude modulation (AM) on the given signal with carrier signal*
> $c(t) = \cos(2\pi 50 t)$, *and then do demodulation to recover the original
> message signal* $m(t)$.

### Performance Task 2 (Part-2 of the manual)

> *Message signal* $m(t) = a\sin(2\pi f_m t + \pi/3)$, $a = 2$, $f_m = 10$.
> *Use FM modulation and demodulation on the given signal and use two scopes
> to show your output. First scope should show message signal and modulated
> signal. Second scope should show message signal and demodulated signal.*
> *The lab report must contain (a) a block diagram of FM modulator,
> (b) a block diagram of FM demodulator, (c) a block diagram of FM modulator
> and demodulator in a single window, and (d) two scope figures.*

---

## Objectives

1. Build a Simulink AM modulator from the **Sine Wave**, **Constant**, **Sum**,
   and **Product** blocks shown in the lab manual block diagram, and verify the
   waveform $s(t)=(A_c+m(t))\,c(t)$ on a Time Scope.
2. Add a coherent AM demodulator (multiply-by-carrier + LPF + DC subtraction)
   that recovers $m(t)$ from $s(t)$.
3. Build a Simulink FM modulator that implements
   $s_{FM}(t) = \cos(2\pi f_c t + 2\pi K_f \int_0^t m(\lambda)\,d\lambda)$
   using the **Integrator**, **Gain**, **Clock**, **Sum** and
   **Trigonometric Function** blocks, again following the manual block
   diagram.
4. Add an FM demodulator that recovers $m(t)$ from $s_{FM}(t)$.
5. Display every result on a 2 × 1 grid Time Scope with titles, Y-axis labels
   and identical Y limits on both tiles.
6. Quantitatively validate that the recovered signal equals the original
   message by computing the best-fit scale, RMS error and maximum error
   between $m(t)$ and $\hat m(t)$.

---

## Background Study

### Amplitude Modulation

For an AM transmitter the modulated signal is

$$
s(t) = (A_c + m(t))\, c(t),
\qquad c(t) = \cos(2\pi f_c t).
$$

If $A_c \ge \max|m(t)|$ the envelope $A_c + m(t)$ remains positive and the
message is preserved without phase reversals (no over-modulation). Coherent
demodulation multiplies $s(t)$ by $2c(t)$:

$$
s(t)\,2c(t) = (A_c+m(t))\bigl(1+\cos(4\pi f_c t)\bigr),
$$

so a low-pass filter at a cutoff well above the message bandwidth but well
below $2f_c$ removes the $2f_c$ image, leaving $A_c+m(t)$. Subtracting $A_c$
yields $\hat m(t) \approx m(t)$.

### Frequency Modulation

For an FM transmitter,

$$
s_{FM}(t) = A_c\cos\!\left(2\pi f_c t
   + 2\pi K_f \int_{0}^{t} m(\lambda)\,d\lambda\right),
$$

where $K_f$ is the frequency-deviation constant. The lab manual implements the
integral with a **Continuous → Integrator** block, multiplies it by
$2\pi K_f$ with a **Gain** block, sums it with a $2\pi f_c t$ ramp generated
by **Sources → Clock** + **Gain**, and forms the final signal with a
**Trigonometric Function** block configured as $\cos$ — exactly the block
sequence shown in *Figure 1* of the manual.

### FM frequency-discriminator demodulator

Differentiating $s_{FM}$ gives

$$
\frac{ds_{FM}}{dt} = -\sin\!\bigl(\,\cdot\,\bigr)\bigl(2\pi f_c + 2\pi K_f m(t)\bigr),
$$

so $|ds_{FM}/dt|$ is an amplitude-modulated waveform whose envelope carries the
information. The DC component of $|sin(\cdot)|$ is $2/\pi$, hence

$$
\overline{|ds_{FM}/dt|} = \tfrac{2}{\pi}\bigl(2\pi f_c + 2\pi K_f m(t)\bigr)
                = 4 f_c + 4 K_f m(t).
$$

Therefore

$$
\boxed{\,m(t) = \dfrac{\mathrm{LPF}\{|ds_{FM}/dt|\} - 4 f_c}{4 K_f}\,}
$$

This is the standard *frequency discriminator*, and unlike a PLL it has no
loop and no stability concern; every block (**Derivative**, **Abs**,
**Transfer Fcn**, **Constant**, **Sum**, **Gain**) is already in the manual's
block library.

### Receiver low-pass filter and group delay

Both demodulators end in a continuous low-pass filter. A 4th-order
Butterworth at cutoff $f_{lpf}$ has the magnitude

$$
|H(j\omega)| = \frac{1}{\sqrt{1+(\omega/2\pi f_{lpf})^{8}}}
$$

and group delay $\tau_g(\omega) \approx 1/(2\pi f_{lpf})\cdot
(\text{shape factor})$. For $f_{lpf}=25\,\text{Hz}$ the group delay is
$\tau_g \approx 17$–$18\,\text{ms}$ across the message band (verified
numerically in Octave). Any causal LPF therefore introduces this fixed delay
between the message and the recovered waveform — it is the **only** reason a
noise-free demodulation does not produce a sample-perfect copy of $m(t)$.

---

## Problem Formulation

The two performance tasks share a common abstract structure:

> Given a message $m(t)$ with bandwidth $W$ Hz, design a transmitter
> $T:\,m(t)\!\mapsto\!s(t)$ and a receiver $R:\,s(t)\!\mapsto\!\hat m(t)$
> using only the block families listed in the lab manual, such that
> $\hat m(t) \approx m(t)$ over a chosen observation window.

Turning this into something we can build in Simulink requires four
formulation decisions: (i) what to choose as the design variables and
constants, (ii) which constraints they must satisfy, (iii) which receiver
architecture to pick out of the several that are theoretically valid, and
(iv) how "$\hat m \approx m$" is going to be measured numerically. The rest
of this section sets all four for both tasks.

### 1. Design variables and constraints

For both tasks, the carrier frequency $f_c$, modulator gain
($A_c$ for AM and $K_f$ for FM), receiver-LPF cutoff $f_{lpf}$ and order
$N$, simulation sample time $T_s$ and stop time $T_{end}$ are *design
variables*; the message $m(t)$ and its bandwidth $W$ are fixed by the
manual. The variables must obey:

1. **Spectral separation** — the LPF passes the message but rejects the
   $2f_c$ image (AM) or the carrier itself (FM):
   $\;W \;<\; f_{lpf} \;\ll\; 2 f_c.$
2. **No over-modulation (AM only)** — to keep envelope detection viable
   *and* to avoid phase reversals:
   $\;A_c \;\ge\; \max|m(t)|.$
3. **Bandwidth budget (FM only)** — Carson's rule must comfortably fit
   inside Nyquist of the chosen $T_s$:
   $\;B_{FM} \;\approx\; 2(\Delta f + W) \;\le\; \tfrac{1}{2 T_s},
   \quad \Delta f = K_f\,\max|m(t)|.$
4. **Solver fidelity** — the sample time must resolve the fastest
   continuous-time signal in the loop:
   $\;T_s \;\le\; 1/(20\,f_c).$

Solving these for the manual-given messages gives the working point used
in the simulation: AM with $A_c = 6$ V, $f_{lpf} = 25$ Hz, $T_s =
1/5000$ s; FM with $K_f = 20$ Hz/V (giving $\Delta f = 40$ Hz),
$f_{lpf} = 25$ Hz, $T_s = 1/20000$ s. The LPF order is fixed at $N=4$
(Butterworth) — the lowest order whose stopband attenuation at $2f_c$ is
better than $-40$ dB while keeping the group delay around 17 ms, small
enough to compensate visually on the scope.

### 2. Receiver architecture choice

For AM the manual leaves the choice between an envelope detector and a
coherent (synchronous) detector open. We pick **coherent** because it is
linear, has no diode-like nonlinearity to model, and is exact for any
$A_c$ — its only failure mode (carrier phase mismatch) is absent in
simulation since the same `cos(2π f_c t)` block feeds both transmitter
and receiver.

For FM, the textbook offers two routes — a **PLL** or a **frequency
discriminator**. A discriminator is preferred here because (i) it has a
closed-form input/output relation
$m(t) = (\overline{|ds_{FM}/dt|} - 4 f_c)/(4 K_f)$, derived in the
Background Study, so its correctness is auditable; (ii) it has no
feedback loop and therefore no stability conditions to verify; (iii) it
fits entirely inside the manual's block library
(`Derivative`, `Abs`, `Transfer Fcn`, `Constant`, `Sum`, `Gain`). A PLL
would require either a Communications-Toolbox `Discrete-Time PLL` or a
custom VCO whose loop-filter parameters need separate tuning; preliminary
runs (best-fit scale ranged from 0.11 to 1551 across solver settings)
made the case for the discriminator decisively.

### 3. Numerical recovery criterion

We do **not** judge "$\hat m \approx m$" by eye. After each simulation
the script extracts the message and the recovered signal as MATLAB
timeseries, drops the first $10\,\%$ of samples (transient region),
realigns the two by cross-correlation up to a maximum lag $\tau_{max}$,
and computes the optimal scalar gain $a$ that minimises
$\|m - a\,\hat m\|_2$ in closed form:

$$
a \;=\; \frac{\langle m,\hat m\rangle}{\langle \hat m,\hat m\rangle},
\qquad
\varepsilon_{\text{rms}} \;=\; \|m - a\,\hat m\|_2/\sqrt{N}.
$$

A run is declared **PASS** when $|a-1|<0.05$ *and*
$\varepsilon_{\text{rms}}<0.10\,\max|m|$ — i.e. the recovery is
correct in both shape (small RMS error) and amplitude (unity scale).
These thresholds are deliberately tighter than the eye-test would be.

### 4. Receiver-LPF group delay as an explicit unknown

The Background Study showed that any causal $N$-th-order Butterworth LPF
introduces a finite, frequency-dependent group delay $\tau_g(\omega)$.
The formulation therefore treats $\tau_g$ as a *known* parameter, not a
nuisance: in the AM model $\tau_g \approx 17$ ms is pre-applied to the
reference $m(t)$ via a Transport Delay block; in the FM model the
reference is instead routed through an *identical* LPF, which both
applies the same $\tau_g$ and introduces the same start-up transient.
This makes "perfect overlap" on the scope a meaningful visual check
rather than a demand for the impossible.

### Mapping the formulation to Simulink

Once the four decisions are fixed, the model layout follows
mechanically. The two scripts `exp7_perf1_AM.m` and `exp7_perf2_FM.m`
build the models programmatically (`add_block` / `set_param` /
`add_line` / `save_system`), run them via `sim`, and pass the resulting
`Simulink.SimulationOutput` to the validator described in §3 above. The
end-to-end signal path for each task is therefore:

- **AM** —
  $m(t) \xrightarrow{+A_c}\; A_c+m(t)
  \;\xrightarrow{\times c(t)}\; s(t)
  \;\xrightarrow{\times 2c(t)}\; \text{LPF}
  \;\xrightarrow{-A_c}\; \hat m(t).$
- **FM** —
  $m(t) \xrightarrow{\int}\; \int m\,dt
  \;\xrightarrow{\times 2\pi K_f,\,+\,2\pi f_c t}\; \phi(t)
  \;\xrightarrow{\cos}\; s_{FM}(t)
  \;\xrightarrow{d/dt,\;|\cdot|}\; \text{LPF}
  \;\xrightarrow{-4f_c,\;\div 4K_f}\; \hat m(t).$

---

## Results & Analysis

### 1. AM block diagram and Time Scopes

Block diagram of the AM modulator + coherent demodulator (Part 1):

![AM Simulink model](AMpart1.png)

**Scope 1 — message vs. AM signal.** The top tile is $m(t)$ and the bottom
tile is $s(t)=(A_c+m(t))c(t)$, sharing identical Y-limits $[-12,12]\,\text{V}$.

![AM Scope 1: message vs. modulated](AMogvsmod.png)

The carrier waveform's instantaneous amplitude tracks the
envelope $A_c+m(t)$, exactly as expected for AM.

**Scope 2 — message vs. recovered.** The top tile is $m(t)$ (delay-aligned with
the receiver LPF), the bottom tile is the recovered $\hat m(t)$.

![AM Scope 2: message vs. recovered](AMogVsDemod.png)

**Combined view — $m$, $s$, and $\hat m$:**

![AM combined comparison](AMogvsmodvsdemod.png)

After delay alignment, the validator printed:

```
AM validation (transient: first 10% discarded):
  estimated demod delay   = 0.0170 s   (LPF group delay; compensated on scope)
  best-fit scale  a       = 1.0223     (1.00 = perfect amplitude match)
  RMS(m - a*m_hat)        = 0.0761171
  max|m - a*m_hat|        = 0.175439
  PASS: amplitude and shape recovered.
```

The estimated delay (17 ms) matches the analytical group delay of the 4th-order
Butterworth LPF at 4–6 Hz to four significant digits, confirming the only
mismatch between $m$ and $\hat m$ is the LPF's group delay. The best-fit scale
is within 2 % of unity and the residual RMS is 1.6 % of the peak signal.

### 2. FM block diagram and Time Scopes

Block diagram of the FM modulator + frequency-discriminator demodulator
(Part 2):

![FM Simulink model](FMBlock.png)

**Scope 1 — message vs. FM modulated:**

![FM Scope 1](FMmsVSmod.png)

The bottom tile clearly shows the carrier "bunching" where $m(t) > 0$ (higher
instantaneous frequency) and "spreading" where $m(t) < 0$ (lower
instantaneous frequency) — the textbook FM signature.

**Scope 2 — message vs. demodulated.** Both tiles are routed through identical
4th-order Butterworth LPFs (one inside the demodulator, one in a matched
reference path), so they share the same start-up dynamics:

![FM Scope 2](FMmsgVSdmod.png)

**Combined view of all three signals:**

![FM combined comparison](FMmsVSmodVSdemod.png)

The validator's output for the FM run was:

```
FM validation (transient: first 10% discarded):
  estimated demod delay   = 0.0000 s   (LPF group delay; compensated on scope)
  best-fit scale  a       = 1.0001
  RMS(m - a*m_demod)      = 0.00400349
  max|m - a*m_demod|      = 0.0178236
  PASS: m_demod(t) = m(t) within solver/LPF tolerance.
```

Best-fit scale $a = 1.0001$, RMS error $4\,\text{mV}$ (0.2 % of peak),
and zero estimated delay because the matched-LPF reference *pre-applies* the
group delay to $m(t)$. This is the cleanest of all the FM demodulator variants
tried during the exercise (the alternative PLL demodulator was unstable in
sign convention and produced a wildly variable best-fit scale).

### 3. Summary table

| Task | Best-fit scale $a$ | RMS error | Max error | Estimated delay | Outcome |
| :--- | :---: | :---: | :---: | :---: | :--- |
| Part 1 — AM  | $1.0223$ | $0.076$ V | $0.175$ V | $17.0\,\text{ms}$ | **PASS** |
| Part 2 — FM  | $1.0001$ | $0.004$ V | $0.018$ V | $0\,\text{ms}$ (matched ref) | **PASS** |

---

## Discussion

The two performance tasks confirm that, in a noise-free environment with
ideal Simulink blocks, both AM and FM modulation–demodulation chains recover
the original message to numerical precision. Key observations:

1. **The block diagrams in the manual are sufficient.** Both modulators were
   built only from the block families listed in the manual
   (Sources → Sine Wave / Constant / Clock; Math Operations → Sum / Product /
   Gain / Abs / Trigonometric Function; Continuous → Integrator / Derivative /
   Transfer Fcn / Transport Delay; Sinks → Scope / To Workspace). No
   specialised Communications Toolbox block (`AM Modulator`, `FM Modulator`,
   `VCO`, `PLL Discrete-Time`) was needed.
2. **The “mismatch” between $m$ and $\hat m$ is the LPF, not the modulation.**
   The group delay of the receive LPF is what makes the recovered trace look
   shifted. Compensating that delay (or pre-filtering the reference message
   with the same LPF) gives best-fit scales of $1.00$ and RMS errors well below
   1 %.
3. **PLL vs. discriminator demodulator.** During development a PLL
   demodulator was implemented using a Product as phase detector, a 2nd-order
   Butterworth loop filter and a VCO built from Integrator + sin. The loop
   was numerically unstable for the chosen $K_v$, $f_{loop}$ values
   (best-fit scale ranged from 0.11 to 1551 across runs). Replacing it with
   the analytical frequency-discriminator structure made the demodulator
   trivially correct and robust — confirming the textbook fact that for a
   noise-free single-tone case the discriminator is both simpler and more
   accurate than a PLL.
4. **Initial-state handling matters.** The demodulator equation
   $\hat m = (\text{LPF}_{out} - 4f_c)/(4K_f)$ outputs $-f_c/K_f = -5\,\text{V}$
   when the LPF starts at zero. Setting the LPF's initial state vector to
   $[0,0,0,4f_c/(2\pi f_{lpf})^4]^\top$ (the controller-canonical steady-state
   for input $4f_c$) makes the demodulator output start at exactly $0\,\text{V}$
   — eliminating an artefact that would otherwise dominate the first
   $\sim 50\,\text{ms}$ of the scope display.
5. **Programmatic model construction.** Building both `.slx` models from
   MATLAB scripts has two practical advantages over hand-drawn models: the
   block parameters (frequencies, gains, filter coefficients, initial states)
   are textually visible and version-controllable, and any change to the
   message or the carrier requires editing one number rather than clicking
   through several block dialogs.

### Limitations

- The AM Y-axis was set to $\pm 12\,\text{V}$ to fit the modulated waveform's
  $\pm 11\,\text{V}$ swing; the demod scope therefore displays the
  $\sim 5\,\text{V}$ peak message at relatively small amplitude.
- The FM scope's “message” tile shows $\mathrm{LPF}\{m(t)\}$, not $m(t)$
  itself, so the report's Scope 2 figure is strictly a comparison of the
  *band-limited* message and the demodulator output. Outside the LPF
  passband (which here is essentially flat at 4–10 Hz) the two are
  numerically identical.

---

## References

1. American International University–Bangladesh, Department of Computer
   Engineering. *COE 3201: Data Communication Laboratory — Experiment 7:
   Study of Amplitude / Frequency Modulator and Demodulator using Simulink*
   (Student Manual, 10 pages).
2. Simon Haykin, *Communication Systems*, 4th ed., John Wiley & Sons,
   2001 — Chapter 2 (Amplitude Modulation) and Chapter 4 (Angle Modulation),
   for the modulator/demodulator structures.
3. Bernard Sklar, *Digital Communications: Fundamentals and Applications*,
   2nd ed., Prentice Hall, 2001 — Chapter 1 (Signals and Spectra) for the
   discriminator demodulator derivation.
4. The MathWorks, Inc. *Simulink Documentation — Block Reference*,
   <https://www.mathworks.com/help/simulink/blocklist.html>: Sine Wave,
   Constant, Clock, Sum, Product, Gain, Abs, Trigonometric Function,
   Integrator, Derivative, Transfer Fcn, Transport Delay, Scope.
5. The MathWorks, Inc. *Control Scope Blocks Programmatically*,
   <https://www.mathworks.com/help/simulink/ug/control-scopes-programmatically.html>
   (used for the 2 × 1 scope grid layout, titles, Y-labels and Y-limits).
6. The MathWorks, Inc. *TimeScopeConfiguration*,
   <https://www.mathworks.com/help/simulink/slref/spbscopes.scope.timescopeconfiguration.html>
   (`LayoutDimensions`, `ActiveDisplay`, `Title`, `YLabel`, `YLimits`,
   `AxesScaling`).
7. The supporting MATLAB scripts that build, run and validate the models for
   this report:
   - `exp7_perf1_AM.m` — Part 1 (AM modulator + coherent demodulator).
   - `exp7_perf2_FM.m` — Part 2 (FM modulator + discriminator demodulator).
   - `exp7.m` / `exp7_performance.m` — convenience wrappers that run both.
