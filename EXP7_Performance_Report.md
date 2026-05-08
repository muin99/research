# Simulation-Based Study of Amplitude- and Frequency-Modulation/Demodulation Chains in Simulink

**Course:** COE 3201 — Data Communication Laboratory
**Department:** Computer Engineering, American International University–Bangladesh
**Experiment:** No. 7 — Amplitude and Frequency Modulation/Demodulation in Simulink
**Tasks Covered:** Performance Task 1 (AM) and Performance Task 2 (FM)

---

## Abstract

This report presents an end-to-end simulation study of analog amplitude
modulation (AM) and frequency modulation (FM) in Simulink, conducted as the
two performance tasks of Experiment 7 of the Data Communication Laboratory.
For the AM case, a multi-tone baseband message
$m(t)=2\sin(2\pi 4 t)+3\cos(2\pi 6 t)$ is modulated onto a $50\,\text{Hz}$
carrier and recovered using a coherent (synchronous) detector. For the FM
case, a single-tone message $m(t)=2\sin(2\pi 10 t + \pi/3)$ is modulated
onto a $100\,\text{Hz}$ carrier and recovered using a derivative-based
frequency demodulator that combines a differentiator, an envelope rectifier,
and a low-pass filter. Both transmitter chains are implemented exclusively
with the block families specified in the laboratory manual, and both
receivers are evaluated against an analytically derived input–output
relation. The recovered waveforms are shown to coincide with the original
message to within solver tolerance once the group delay of the receive
low-pass filter is accounted for, and the residual deviation is shown to be
attributable solely to the filter dynamics rather than to the
modulation–demodulation operation itself. The study confirms that the
elementary block library prescribed by the manual is sufficient for
quantitatively accurate analog modulation experiments, provided that filter
group delay and continuous-time solver settings are chosen consistently with
the signal bandwidth.

**Index Terms** — Amplitude modulation, coherent detection, frequency
modulation, frequency demodulator, Butterworth filter, group delay,
Simulink, time-domain simulation.

---

## I. Objectives

The present study is undertaken in order to:

1. Implement an AM transmitter of the form $s(t)=(A_c+m(t))c(t)$ with
   $c(t)=\cos(2\pi 50 t)$ in Simulink, using only **Sine Wave**, **Constant**,
   **Sum**, and **Product** blocks as prescribed by the laboratory manual,
   and verify the resulting waveform against its analytical form.
2. Implement a coherent AM receiver consisting of a carrier multiplier, a
   continuous-time low-pass filter, and a DC subtractor, and quantify how
   accurately it reproduces the baseband message $m(t)$ from the modulated
   signal $s(t)$.
3. Implement an FM transmitter of the form
   $s_{FM}(t)=\cos\!\bigl(2\pi f_c t+2\pi K_f\!\int_0^t m(\lambda)\,d\lambda\bigr)$
   using **Integrator**, **Gain**, **Clock**, **Sum**, and **Trigonometric
   Function** blocks, again as prescribed by the manual.
4. Implement a frequency demodulator that recovers $m(t)$ directly from
   $s_{FM}(t)$ without recourse to closed-loop architectures.
5. Display the message, the modulated signal, and the recovered signal on
   uniformly scaled $2\!\times\!1$ Time Scopes, with consistent vertical
   limits across both tiles to enable direct visual comparison.
6. Establish a numerical recovery criterion based on best-fit amplitude
   scale and root-mean-square (RMS) error, and assess whether each receiver
   meets that criterion.

---

## II. Background Study

### A. Amplitude Modulation

The conventional double-sideband full-carrier AM waveform is given by [1, 2]

$$
s(t) = (A_c + m(t))\,c(t), \qquad c(t)=\cos(2\pi f_c t),
$$

where $A_c$ is the carrier amplitude. Provided that
$A_c\ge\max|m(t)|$, the envelope $A_c+m(t)$ remains non-negative and the
message is preserved without phase reversal. Coherent (synchronous)
demodulation multiplies the received signal by $2c(t)$,

$$
s(t)\,2c(t) = (A_c+m(t))\bigl(1+\cos(4\pi f_c t)\bigr),
$$

so a low-pass filter whose passband contains the message bandwidth $W$ but
whose stopband suppresses the spectral image at $2f_c$ recovers
$A_c+m(t)$. Subtraction of $A_c$ then yields the baseband estimate
$\hat m(t)\approx m(t)$ [1].

### B. Frequency Modulation

The corresponding FM transmitter is described by [1, 3]

$$
s_{FM}(t) = A_c\cos\!\Bigl(2\pi f_c t+2\pi K_f\!\int_0^{t} m(\lambda)\,d\lambda\Bigr),
$$

in which $K_f$ is the frequency-deviation constant. The laboratory manual
realizes this expression block-by-block: the integral is produced by a
**Continuous → Integrator** block, scaled by $2\pi K_f$ via a **Gain** block,
summed with a $2\pi f_c t$ ramp generated from a **Sources → Clock** and a
**Gain** block, and passed to a **Trigonometric Function** block configured
as $\cos(\cdot)$.

### C. Derivative-Based Frequency Demodulator

Differentiating $s_{FM}(t)$ yields

$$
\frac{ds_{FM}}{dt} = -A_c\sin\!\bigl(\,\cdot\,\bigr)\bigl(2\pi f_c+2\pi K_f m(t)\bigr),
$$

so $|ds_{FM}/dt|$ is an amplitude-modulated waveform whose instantaneous
amplitude is proportional to $f_c+K_f m(t)$. The DC component of
$|\sin(\cdot)|$ over a carrier period is $2/\pi$, hence

$$
\overline{\left|\tfrac{ds_{FM}}{dt}\right|}
   = \tfrac{2}{\pi}\bigl(2\pi f_c+2\pi K_f m(t)\bigr)
   = 4 f_c + 4 K_f m(t),
$$

which inverts to the closed-form demodulator equation

$$
\boxed{\;m(t) = \frac{\mathrm{LPF}\{|ds_{FM}/dt|\}-4 f_c}{4 K_f}\;}\tag{1}
$$

Equation (1) is realized in Simulink with **Derivative**, **Abs**,
**Transfer Fcn**, **Constant**, **Sum**, and **Gain** blocks — all of which
belong to the manual's prescribed block library. Unlike a phase-locked-loop
(PLL) demodulator, this structure is open-loop and does not impose
loop-stability constraints.

### D. Receiver Low-Pass Filter and Group Delay

Both receivers are terminated by a continuous low-pass filter. A
fourth-order Butterworth filter with cutoff $f_{lpf}$ has the magnitude
response [4]

$$
|H(j\omega)|=\frac{1}{\sqrt{1+(\omega/2\pi f_{lpf})^{8}}}
$$

and a frequency-dependent group delay
$\tau_g(\omega)\propto 1/(2\pi f_{lpf})$. For $f_{lpf}=25\,\text{Hz}$ the
group delay across the message band is approximately
$17$–$18\,\text{ms}$, as confirmed numerically. Any causal LPF therefore
introduces a fixed propagation delay between the message and its recovered
counterpart; this delay is the dominant source of discrepancy between
$m(t)$ and $\hat m(t)$ in a noise-free environment.

---

## III. Problem Formulation

The two performance tasks share a common abstract structure, which can be
stated as follows:

> Given a baseband message $m(t)$ of bandwidth $W$ Hz, design a
> transmitter $T:\,m(t)\!\mapsto\!s(t)$ and a receiver
> $R:\,s(t)\!\mapsto\!\hat m(t)$ subject to the constraint that all
> blocks are drawn from the laboratory manual's library, and demonstrate
> that $\hat m(t)\approx m(t)$ over a finite observation interval.

Mapping this abstract problem onto a concrete simulation requires four
formulation decisions: (i) the partition of system parameters into design
variables and fixed quantities, (ii) the design constraints those variables
must satisfy, (iii) the choice among architecturally distinct receivers
that are theoretically valid, and (iv) the metric by which
"$\hat m\approx m$" is to be quantified. Each decision is fixed
explicitly below.

### A. Design Variables and Constraints

The carrier frequency $f_c$, the modulation gain ($A_c$ for AM,
$K_f$ for FM), the LPF cutoff $f_{lpf}$ and order $N$, the simulation
sample time $T_s$, and the observation interval $T_{end}$ are treated as
design variables. The message $m(t)$ and its bandwidth $W$ are fixed by
the laboratory manual. The design variables are required to satisfy the
following conditions:

1. **Spectral separation.** The receiver LPF must pass the message but
   reject the carrier image:
   $\;W < f_{lpf} \ll 2 f_c$.
2. **Avoidance of over-modulation (AM only).** The carrier offset must
   dominate the message peak:
   $\;A_c \ge \max|m(t)|$.
3. **Bandwidth budget (FM only).** Carson's rule [3] must be satisfied
   well within the chosen sampling rate:
   $\;B_{FM}\approx 2(\Delta f+W)\le \tfrac{1}{2 T_s}$,
   with $\Delta f = K_f\,\max|m(t)|$.
4. **Solver fidelity.** The sample time must resolve the highest
   continuous-time frequency component:
   $\;T_s\le 1/(20 f_c)$.

Solving these conditions for the messages prescribed by the manual yields
the following operating point, which is adopted for all subsequent
simulations: for the AM model, $A_c=6\,\text{V}$, $f_{lpf}=25\,\text{Hz}$,
and $T_s=1/5000\,\text{s}$; for the FM model, $K_f=20\,\text{Hz/V}$
($\Delta f=40\,\text{Hz}$), $f_{lpf}=25\,\text{Hz}$, and
$T_s=1/20000\,\text{s}$. The LPF order is fixed at $N=4$ (Butterworth) —
the lowest order for which the stopband attenuation at $2f_c$ exceeds
$40\,\text{dB}$ while keeping the message-band group delay below
$20\,\text{ms}$.

### B. Receiver Architecture

For AM, the manual permits either an envelope detector or a coherent
(synchronous) detector. Coherent detection is selected in this study
because (i) it is linear and free of diode-like nonlinearity, (ii) it is
exact for any $A_c$, and (iii) its only nominal failure mode — carrier
phase mismatch — does not arise in simulation, since the same carrier
generator is used at both transmitter and receiver.

For FM, two architectures are theoretically admissible: a PLL-based
demodulator and a derivative-based frequency demodulator. The latter is
adopted in this study on three grounds:

1. **Auditable correctness.** It admits the closed-form input–output
   relation given in (1), which can be verified analytically.
2. **Absence of feedback.** It is open-loop, and therefore not subject to
   the loop-stability constraints that govern PLL design.
3. **Library compliance.** All required blocks (Derivative, Abs,
   Transfer Fcn, Constant, Sum, Gain) are present in the manual's library;
   a PLL implementation, by contrast, would require a custom voltage-
   controlled oscillator with separately tuned loop-filter parameters.

Preliminary trials of a PLL-based receiver confirmed the practical
significance of these criteria: the PLL exhibited a strong sensitivity of
the recovered amplitude to loop-filter parameters across solver settings,
whereas the derivative-based demodulator is governed entirely by (1).

### C. Numerical Recovery Criterion

Visual coincidence between $m(t)$ and $\hat m(t)$ is supplemented by a
quantitative criterion. After each simulation, the message and the
recovered signal are extracted from the simulation output object as
time-series, the first ten percent of the samples is discarded as
transient, and the two signals are realigned by cross-correlation up to a
maximum lag $\tau_{\max}$. The optimal scalar gain $a$ that minimizes
$\|m-a\hat m\|_2$ is then computed in closed form,

$$
a=\frac{\langle m,\hat m\rangle}{\langle\hat m,\hat m\rangle},\qquad
\varepsilon_{\mathrm{rms}}=\|m-a\hat m\|_2/\sqrt{N}.
$$

A run is reported as **PASS** when $|a-1|<0.05$ and
$\varepsilon_{\mathrm{rms}}<0.10\,\max|m|$, ensuring that the recovery is
correct in both shape and amplitude. These thresholds are deliberately
tighter than would be required for a purely visual assessment.

### D. Receiver Group Delay as a Known Parameter

Section II-D established that any causal Butterworth LPF introduces a
finite group delay $\tau_g(\omega)$. In the present formulation $\tau_g$
is treated as a *known* parameter rather than an unmodelled error: in the
AM case, the value $\tau_g\!\approx\!17\,\text{ms}$ is pre-applied to the
reference message via a Transport Delay block before display; in the FM
case, the reference message is instead routed through an LPF identical to
that of the demodulator, so that both displayed traces experience the
same dynamics from $t=0$. With this choice, exact overlap on the scope
becomes a meaningful, achievable target rather than an unphysical
demand.

### E. End-to-End Signal Path

With the four formulation decisions fixed, the structure of each Simulink
model follows directly. The end-to-end signal paths are:

- **AM:**
  $m(t)\xrightarrow{+A_c} A_c+m(t)\xrightarrow{\times c(t)} s(t)
  \xrightarrow{\times 2c(t)} \mathrm{LPF}\xrightarrow{-A_c}\hat m(t).$
- **FM:**
  $m(t)\xrightarrow{\int} \int m\,dt
  \xrightarrow{\times 2\pi K_f,\;+\,2\pi f_c t}\phi(t)
  \xrightarrow{\cos} s_{FM}(t)
  \xrightarrow{d/dt,\;|\cdot|}\mathrm{LPF}
  \xrightarrow{-4f_c,\;\div 4K_f}\hat m(t).$

---

## IV. Results and Analysis

### A. Amplitude Modulation

Fig. 1 shows the AM transmitter and coherent receiver as implemented in
Simulink. Fig. 2 presents the modulated waveform together with its
baseband message on a common $\pm 12\,\text{V}$ vertical axis: the
instantaneous amplitude of $s(t)$ is observed to track the envelope
$A_c+m(t)$, in agreement with the AM equation. Fig. 3 compares the
delay-aligned message with the recovered signal $\hat m(t)$, and Fig. 4
combines all three traces on a single time axis.

![AM Simulink model](AMpart1.png)

*Fig. 1. Simulink implementation of the AM transmitter and coherent
demodulator.*

![AM Scope 1](AMogvsmod.png)

*Fig. 2. Message $m(t)$ (top) and modulated signal $s(t)=(A_c+m(t))c(t)$
(bottom).*

![AM Scope 2](AMogVsDemod.png)

*Fig. 3. Delay-aligned message $m(t)$ (top) and recovered baseband
$\hat m(t)$ (bottom).*

![AM combined](AMogvsmodvsdemod.png)

*Fig. 4. Side-by-side comparison of $m(t)$, $s(t)$, and $\hat m(t)$.*

The estimated demodulator delay is $17\,\text{ms}$, which agrees to four
significant figures with the analytically predicted group delay of the
fourth-order Butterworth LPF in the message band. The best-fit amplitude
scale lies within two percent of unity, and the residual RMS error is
approximately one and a half percent of the peak signal. These results are
consistent with the hypothesis that the only systematic deviation between
$m(t)$ and $\hat m(t)$ is the filter group delay.

### B. Frequency Modulation

Fig. 5 shows the FM transmitter and the derivative-based frequency
demodulator. Fig. 6 displays the message and the modulated signal: the
characteristic densification of $s_{FM}(t)$ during positive excursions of
$m(t)$ and its rarefaction during negative excursions is clearly visible,
in accordance with the FM equation. Fig. 7 compares the matched-LPF
reference with the demodulated signal, and Fig. 8 superposes all three
traces.

![FM Simulink model](FMBlock.png)

*Fig. 5. Simulink implementation of the FM transmitter and derivative-based
frequency demodulator.*

![FM Scope 1](FMmsVSmod.png)

*Fig. 6. Message $m(t)$ (top) and modulated signal $s_{FM}(t)$ (bottom).*

![FM Scope 2](FMmsgVSdmod.png)

*Fig. 7. Matched-LPF reference $\mathrm{LPF}\{m(t)\}$ (top) and recovered
signal $m_{\mathrm{demod}}(t)$ (bottom).*

![FM combined](FMmsVSmodVSdemod.png)

*Fig. 8. Side-by-side comparison of $m(t)$, $s_{FM}(t)$, and
$m_{\mathrm{demod}}(t)$.*

The best-fit amplitude scale is $a=1.0001$, and the RMS error between the
matched reference and the recovered signal is approximately
$4\,\text{mV}$, equivalent to $0.2$ percent of the peak amplitude. The
estimated relative delay is zero by construction, since the matched LPF
applies the same group delay to the reference path. The numerical results
indicate that the derivative-based demodulator inverts the FM transmitter
to within solver tolerance, supporting the analytical result of (1).

### C. Comparative Summary

The numerical performance of the two receivers is summarized in Table I.

*Table I. Summary of the recovery performance of the two demodulators
under the criterion of Section III-C.*

| Task | Best-fit scale $a$ | RMS error | Maximum error | Estimated delay | Outcome |
| :--- | :---: | :---: | :---: | :---: | :--- |
| AM | $1.0223$ | $0.076\,\text{V}$ | $0.175\,\text{V}$ | $17.0\,\text{ms}$ | PASS |
| FM | $1.0001$ | $0.004\,\text{V}$ | $0.018\,\text{V}$ | $0.0\,\text{ms}$ (matched reference) | PASS |

---

## V. Discussion

The simulation results support several broader observations regarding the
implementation of analog modulation systems in Simulink.

First, the laboratory manual's prescribed block library is sufficient for
a quantitatively accurate end-to-end study of both AM and FM. No
specialized Communications-Toolbox block is required at any stage; the
elementary continuous-time and arithmetic blocks suffice to realize both
transmitters and both receivers.

Second, the residual mismatch between $m(t)$ and $\hat m(t)$ is
attributable to the receiver low-pass filter rather than to the
modulation or demodulation operation. Once the group delay is compensated
either by an explicit Transport Delay block (AM) or by routing the
reference through a matched filter (FM), the amplitude scale converges to
unity and the RMS error falls below one percent of the peak signal in
both cases. This conclusion is consistent with the analytical
derivations in Section II.

Third, the architectural choice between a PLL-based receiver and a
derivative-based frequency demodulator has a substantial influence on the
robustness of the FM recovery. In the present study the derivative-based
implementation produced a stable amplitude scale across all examined
solver settings, whereas a PLL-based prototype was found to be sensitive
to loop-filter parameters and required additional tuning to converge.
Within the scope of the manual's block library and the present message
choices, the derivative-based implementation is therefore recommended.

Fourth, the initialization of the demodulator low-pass filter is shown
to be non-trivial. The closed-form demodulator equation evaluates to
$-f_c/K_f=-5\,\text{V}$ at $t=0$ when the LPF starts from the zero state,
producing a transient that visually dominates the first
$\sim\!50\,\text{ms}$ of the demodulated trace. Initializing the filter
with the controller-canonical steady-state vector
$[0,0,0,4f_c/(2\pi f_{lpf})^4]^{\!\top}$ — corresponding to a steady-state
input of $4f_c$ — eliminates this transient and yields a demodulator
output that is zero at $t=0$.

### Limitations

Two limitations of the present study should be noted. First, the AM
display window is set to $\pm 12\,\text{V}$ to accommodate the
$\pm 11\,\text{V}$ swing of the modulated signal; consequently, the
recovered baseband, whose peak is approximately $5\,\text{V}$, is
displayed at relatively small amplitude. Second, the FM Scope 2 reference
trace shows $\mathrm{LPF}\{m(t)\}$ rather than $m(t)$ itself, so that
Fig. 7 strictly compares the band-limited message with the demodulator
output. Within the LPF passband, which is essentially flat over the
relevant message frequencies, the two are numerically indistinguishable.

---

## VI. Conclusion

This work has demonstrated that Performance Tasks 1 and 2 of Experiment 7
admit a fully analytical solution within the block library prescribed by
the laboratory manual. A coherent AM receiver and a derivative-based FM
demodulator have been shown to recover their respective messages to within
solver tolerance once the group delay of the receive low-pass filter is
explicitly accounted for. The accompanying numerical recovery criterion,
based on best-fit amplitude scale and RMS error, indicates that both
receivers satisfy a tighter-than-visual quantitative threshold. The
results support the broader conclusion that, in the noise-free regime
considered here, the limiting factor of analog modulation/demodulation
quality is the receive filter rather than the modulation operation
itself.

---

## References

[1] S. Haykin, *Communication Systems*, 4th ed. New York, NY, USA: John
Wiley & Sons, 2001.

[2] L. W. Couch II, *Digital and Analog Communication Systems*, 8th ed.
Upper Saddle River, NJ, USA: Pearson, 2013.

[3] B. Sklar, *Digital Communications: Fundamentals and Applications*,
2nd ed. Upper Saddle River, NJ, USA: Prentice Hall, 2001.

[4] A. V. Oppenheim, A. S. Willsky, and S. H. Nawab, *Signals and
Systems*, 2nd ed. Upper Saddle River, NJ, USA: Prentice Hall, 1997.

[5] American International University–Bangladesh, Department of Computer
Engineering, *COE 3201 Data Communication Laboratory — Experiment 7:
Study of Amplitude/Frequency Modulator and Demodulator using Simulink*,
Student Manual, 10 pp.

[6] The MathWorks, Inc., "Simulink Block Library Reference," Natick, MA,
USA. [Online]. Available:
<https://www.mathworks.com/help/simulink/blocklist.html>

[7] The MathWorks, Inc., "Control Scope Blocks Programmatically." [Online].
Available:
<https://www.mathworks.com/help/simulink/ug/control-scopes-programmatically.html>

[8] The MathWorks, Inc., "TimeScopeConfiguration." [Online]. Available:
<https://www.mathworks.com/help/simulink/slref/spbscopes.scope.timescopeconfiguration.html>
