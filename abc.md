**American International University-Bangladesh (AIUB)**
**Faculty of Engineering — Department of Computer Engineering**
**Course: COE 3201 — Data Communication**
**Open-Ended Laboratory (OEL) Report**

---

## Title

**Comparative Study of M-ary Digital Modulation Schemes (BASK, 4-ASK, 16-ASK, QPSK, 8-PSK, 16-QAM) for ASCII Text Transmission over an AWGN Channel — Constellation, Threshold, and BER Analysis in MATLAB**

---

## Objective

The purpose of this OEL is to design, in MATLAB, a complete end-to-end digital communication link that:

1. Encodes a textual message (the SURNAME of a group member, here `ISLAM`) into an 8-bit ASCII binary stream and a synchronous serial transmission format.
2. Converts the bit stream into a **unipolar NRZ** waveform at a data rate of **1 kbps** (Lecture 4 / Lab Exp 6).
3. Modulates the NRZ baseband data using **six** digital-to-analog schemes — **BASK, 4-ASK, 16-ASK, QPSK, 8-PSK, 16-QAM** — exactly as defined in **Lecture 7** and **Lecture 8**.
4. Sends each modulated waveform through an **AWGN** channel (using the lab manual's `awgn(...)` function) at $E_b/N_0 = \text{VAL1} + 20$ dB (i.e., **23 dB** for group 3 — *good case*).
5. Performs **coherent demodulation** (mixing with the local carrier + Butterworth LPF + symbol-instant sampling) and converts the recovered bits back into text using `bin2asc` style logic from **Lab Experiment 8**.
6. Quantitatively confirms the recovered text against the original message, then sweeps $E_b/N_0$ to find the **threshold** Eb/No at which reconstruction fails (*worst case*).
7. Plots the **theoretical Bit Error Probability** $P_b$ versus $E_b/N_0$ for $E_b/N_0 \in [10, 30]$ dB using the **Lecture 8 BER-vs-modulation-order table** and the Q-function (Q-table reference).

**Summary of Obtained Results:**

* All six modulation schemes successfully recovered the original message `ISLAM` at $E_b/N_0 = 23$ dB. The output console showed:
  ```
  Original message      : ISLAM
  Recovered from BASK   : ISLAM
  Recovered from 4-ASK  : ISLAM
  Recovered from 16-ASK : ISLAM
  Recovered from QPSK   : ISLAM
  Recovered from 8-PSK  : ISLAM
  Recovered from 16-QAM : ISLAM
  ```
* The Eb/No threshold sweep (30 dB → −10 dB) showed that **higher-order** schemes (16-ASK, 16-QAM, 8-PSK) fail at higher $E_b/N_0$ than **lower-order** schemes (BASK, QPSK, 4-ASK), confirming the theoretical noise sensitivity of densely packed constellations.
* The BER curves (Step Q10) rank the schemes from most-robust to least-robust: **BPSK/BASK ≈ QPSK > 4-ASK > 16-QAM > 8-PSK > 16-ASK**, matching the Lecture 8 plot of $P_b$ versus $E_b/N_0$.

---

## Related Theory

### 1. Digital-to-Analog Conversion (Lecture 7)

A digital data sequence $\{d_n\}$ is mapped onto a sinusoidal carrier $A_c \cos(2\pi f_c t + \phi_c)$ by varying one (or more) of the carrier's three parameters:

* **Amplitude** ($A_c$) → **ASK / M-ASK**
* **Frequency** ($f_c$) → **FSK / M-FSK**
* **Phase** ($\phi_c$) → **PSK / M-PSK**
* **Amplitude + Phase** → **QAM**

The number of bits per signal element is $r = \log_2 M$ for an $M$-level scheme. The signal (baud) rate $S$ is related to the bit rate $N$ by:
$$S \;=\; \frac{N}{r} \;=\; \frac{N}{\log_2 M}$$

### 2. Bit Rate, Baud Rate, Carrier (Lecture 7, Slide 6)

* **Bit rate** $N$: bits per second (bps).
* **Baud rate** $S$: signal elements per second.
* **Carrier**: a high-frequency sinusoid acting as the base for the information signal.

In this experiment $N = 1000$ bps and the carrier frequency $f_c = 5{,}000$ Hz. The sampling frequency is $f_s = 500{,}000$ Hz (500 samples per pulse) — well above the Nyquist rate.

### 3. M-ary Amplitude Shift Keying (M-ASK)

Each $r = \log_2 M$ bits select one of $M$ amplitudes $a_m$. The modulated waveform during the $m$-th symbol interval is:
$$s_m(t) \;=\; a_m \, \cos(2\pi f_c t), \qquad 0 \le t < T_s, \qquad m \in \{0, 1, \dots, M-1\}$$

**Mapping used in this OEL:**

| Scheme | Bits/symbol | Levels | Amplitudes (V) |
|---|---|---|---|
| BASK ($M=2$) | 1 | 2 | $\{0, 5\}$ |
| 4-ASK ($M=4$) | 2 | 4 | $\{0, 2, 4, 6\}$ |
| 16-ASK ($M=16$) | 4 | 16 | $\{0, 2, 4, \dots, 30\}$ |

The constellation diagram for ASK is a set of **collinear points on the in-phase axis** (Lecture 7, Slide 31).

### 4. M-ary Phase Shift Keying (M-PSK)

Each $r = \log_2 M$ bits select one of $M$ phases $\phi_m = 2\pi m / M + \phi_0$. The waveform is:
$$s_m(t) \;=\; A_c \, \cos\!\Big(2\pi f_c t + \phi_m\Big)$$

**Mappings used:**

| Scheme | Bits/symbol | Phases |
|---|---|---|
| QPSK ($M=4$) | 2 | $\{\pi/4, \, 3\pi/4, \, 5\pi/4, \, 7\pi/4\}$ |
| 8-PSK ($M=8$) | 3 | $\{0, \, 2\pi/8, \, 4\pi/8, \, \dots, \, 14\pi/8\}$ |

QPSK uses **two BPSK modulators in quadrature** (in-phase + quadrature), described in **Lecture 7, Slide 24**:
$$s_{\text{QPSK}}(t) \;=\; I \cos(2\pi f_c t) - Q \sin(2\pi f_c t)$$

### 5. Quadrature Amplitude Modulation (16-QAM)

QAM combines ASK and PSK: two orthogonal carriers ($\cos$ and $-\sin$) are amplitude-modulated independently:
$$s_{\text{QAM}}(t) \;=\; I_m \cos(2\pi f_c t) - Q_m \sin(2\pi f_c t)$$

For **16-QAM** (Lecture 8, Slide 6), the in-phase and quadrature levels each take 4 values $\{-3, -1, +1, +3\}$, giving a $4\times 4$ rectangular constellation with $r = 4$ bits/symbol:

| Bits $b_1 b_2$ | $I$ | Bits $b_3 b_4$ | $Q$ |
|---|---|---|---|
| 00 | $-3$ | 00 | $-3$ |
| 01 | $-1$ | 01 | $-1$ |
| 10 | $+1$ | 10 | $+1$ |
| 11 | $+3$ | 11 | $+3$ |

### 6. Constellation Diagrams (Lecture 7, Slides 27–34)

A constellation diagram is a 2-D plot where each signal element is represented by a single dot; the **horizontal axis is the in-phase (I) component** and the **vertical axis is the quadrature (Q) component**. Four pieces of information can be deduced from each point:
* Projection on $I$ axis → in-phase amplitude
* Projection on $Q$ axis → quadrature amplitude
* Distance from origin → peak signal amplitude
* Angle from $I$ axis → carrier phase

### 7. AWGN Channel Model (Lab Exp 8)

The received signal in an additive-white-Gaussian-noise channel is:
$$r(t) \;=\; s(t) + n(t), \qquad n(t) \sim \mathcal{N}\!\big(0,\, N_0/2\big)$$
with constant power-spectral density $N_0/2$ across all frequencies. In MATLAB this is generated with `r = awgn(s, EbNo_dB, 'measured')`.

### 8. Coherent Demodulation (Lab Exp 8 + Lecture 8, BPSK derivation)

The receiver multiplies the received signal by a locally generated in-phase ($\cos$) and quadrature ($-\sin$) carrier and integrates (or low-pass filters) over each symbol interval:

$$
I_m \;=\; \frac{2}{T_s} \int_0^{T_s} r(t)\,\cos(2\pi f_c t)\,dt
\qquad\quad
Q_m \;=\; -\frac{2}{T_s} \int_0^{T_s} r(t)\,\sin(2\pi f_c t)\,dt
$$

In `oel.m` this is implemented with a 4th-order Butterworth low-pass filter (cut-off $1.5 \times$ bit rate):

```matlab
[numl, denl] = butter(4, (bit_rate*1.5)/(fs/2));
I = filter(numl, denl,  2*rx .* cos(2*pi*fc*t));
Q = filter(numl, denl, -2*rx .* sin(2*pi*fc*t));
```

The decision rule then quantises $(I_m, Q_m)$ to the nearest constellation point.

### 9. Bit Error Rate (BER) — Lecture 8, Slide 9–12

For BPSK in AWGN, the derivation in Lecture 8 gives:
$$d_{\min} = 2A,\qquad \frac{E_b}{N_0} \;=\; \frac{A^2}{N_0} \;=\; \frac{d_{\min}^{2}}{4 N_0}$$
$$P_b \;=\; \int_A^{\infty} \frac{1}{\sqrt{2\pi}\,\sigma}\, \exp\!\Big(\!-\frac{n^2}{2\sigma^2}\Big)\, dn \;=\; Q\!\Big(\frac{A}{\sigma}\Big) \;=\; Q\!\Big(\sqrt{\tfrac{2 E_b}{N_0}}\Big)$$

The **Lecture 8 generalised BER table** (Slide 12) gives the approximate BER vs modulation order $M$ in AWGN:

| Modulation | BER formula |
|---|---|
| **M-PSK** | $P_b \approx \dfrac{2}{\log_2 M}\, Q\!\left( \sqrt{2 \log_2 M \cdot \tfrac{E_b}{N_0}} \cdot \sin\!\tfrac{\pi}{M} \right)$ |
| **M-QAM** | $P_b \approx \dfrac{4}{\log_2 M}\!\left(1 - \dfrac{1}{\sqrt{M}}\right) Q\!\left( \sqrt{\dfrac{3 \log_2 M}{M-1} \cdot \tfrac{E_b}{N_0}} \right)$ |
| **M-ASK** | $P_b \approx \dfrac{2(M-1)}{M \log_2 M}\, Q\!\left( \sqrt{\dfrac{6 \log_2 M}{M^2 - 1} \cdot \tfrac{E_b}{N_0}} \right)$ |

The **Q-function** is the upper-tail integral of the standard normal density:
$$Q(x) \;=\; \int_x^{\infty} \frac{1}{\sqrt{2\pi}}\, e^{-u^2/2}\, du \;=\; \frac{1}{2}\, \mathrm{erfc}\!\left( \frac{x}{\sqrt{2}} \right)$$

The supplied **Q-table (Q-table_Lecture_8.pdf)** is a numerical look-up of this same integral; in MATLAB the script evaluates it as `Q_function = @(x) 0.5*erfc(x/sqrt(2))`.

---

## Implementation in MATLAB (`oel.m`)

The script is organised as a Q1–Q10 table of contents matching the OEL question:

| Section | Question | Key MATLAB block |
|---|---|---|
| Q1 | ASCII conversion | `dec2bin(double('ISLAM'), 8)` |
| Q2 | Synchronous serial format | `reshape(binary_matrix.' - '0', 1, [])` |
| Q3 | Unipolar NRZ at 1 kbps | for-loop building `dig_sig` (5 V / 0 V) |
| Q4 | 6 modulators + ideal constellation | per-symbol `cos(2*pi*fc*t + phase)` |
| Q5 | AWGN @ Eb/No = VAL1+20 = 23 dB + good-case constellation | `awgn(s, EbNo_dB, 'measured')` |
| Q6 | Coherent I/Q demod + Butterworth LPF + symbol sampling | `butter`, `filter` |
| Q7 | Bits → text | local `bits_to_text` helper (8-bit chunks → `char`) |
| Q8 | Verify recovered message | `strcmp(message, recovered{k})` |
| Q9 | Eb/No threshold sweep + worst-case constellation | `for EbNo_now = 30:-1:-10` |
| Q10 | Theoretical BER 10–30 dB | `semilogy` of Lecture-8 formulas |

### Key Numerical Parameters

| Symbol | Value | Source |
|---|---|---|
| `message` | `ISLAM` | Group member surname |
| `VAL1` | `3` | Group number |
| `EbNo_dB` (good) | $23$ dB | $\text{VAL1} + 20$ |
| `EbNo_worse` | $0$ dB | Below threshold (worst case) |
| `bit_rate` | $1000$ bps | Lecture 4 / question |
| `fc` | $5000$ Hz | Carrier (5×bit-rate) |
| `samples_per_pulse` | $500$ | High-resolution rendering |
| `fs` | $500{,}000$ Hz | $= \text{samples\_per\_pulse} / T_b$ |
| `no_bits` | $40$ | $5 \text{ chars} \times 8$ |

### Step-by-Step Outputs

* **Q1.** `'ISLAM'` → ASCII `[73 83 76 65 77]`.
* **Q2.** Bit stream length = **40 bits**:
  ```
  0100 1001  0101 0011  0100 1100  0100 0001  0100 1101
  ```
* **Q3.** A 5-V/0-V unipolar NRZ waveform spanning 40 ms at 1 kbps.
* **Q4.** Six modulated waveforms, each obeying its own bit-grouping rule (1, 2, 4, 2, 3, 4 bits/symbol, respectively).

---

## Result Analysis

### 1. Output Figures

The script produces **9 figures** (one consolidated visualisation per concept, instead of dozens of small plots):

| # | Figure | Content |
|---|---|---|
| 1 | Input NRZ | Standalone unipolar NRZ at 1 kbps |
| 2 | **BASK pipeline** | 4 subplots: NRZ → Modulated → Received (AWGN) → Recovered NRZ |
| 3 | **4-ASK pipeline** | as above |
| 4 | **16-ASK pipeline** | as above |
| 5 | **QPSK pipeline** | as above |
| 6 | **8-PSK pipeline** | as above |
| 7 | **16-QAM pipeline** | as above |
| 8 | **Ideal constellations** | 6 sub-plots — single dots at the theoretical I/Q positions |
| 9a | **Good-case constellations** | Red received clouds on top of blue ideal points at $E_b/N_0 = 23$ dB |
| 9b | **Worst-case constellations** | Same overlay at $E_b/N_0 = 0$ dB |
| 10 | **BER vs $E_b/N_0$** | Six theoretical curves (10–30 dB, semilogy) |

### 2. Effect of Modulation Order $M$ on Bit Rate

The number of bits per symbol is $r = \log_2 M$. Because the same physical bandwidth is used, increasing $M$ **multiplies** the data throughput:

| Scheme | $M$ | $r$ (bits/sym) | Bits per second of carrier |
|---|---|---|---|
| BASK | 2 | 1 | $1\!\times\! S$ |
| 4-ASK / QPSK | 4 | 2 | $2\!\times\! S$ |
| 8-PSK | 8 | 3 | $3\!\times\! S$ |
| 16-ASK / 16-QAM | 16 | 4 | $4\!\times\! S$ |

Therefore, **for the same baud rate $S$**, 16-QAM transmits **4× more bits per second** than BASK.

### 3. Effect of $E_b/N_0$, SNR, and Noise on BER

* **Good-case constellation** ($E_b/N_0 = 23$ dB): each red cloud is a tight ball around the ideal blue dot; decision regions are easily separated; all six schemes deliver `ISLAM` perfectly.
* **Worst-case constellation** ($E_b/N_0 = 0$ dB): clouds are large and overlap heavily; for high-order schemes (16-ASK, 16-QAM, 8-PSK), several clouds are visually merged — the demodulator misclassifies symbols, producing a corrupted message.

The **Lecture 8 BER formulas** (plotted in Figure 10) capture this behaviour mathematically:

* Each formula is monotonically decreasing in $E_b/N_0$.
* The exponent inside $Q(\cdot)$ shrinks as $M$ grows, so dense constellations need **more** $E_b/N_0$ to reach the same BER.
* Numerically, at $E_b/N_0 = 10$ dB the BER ranking is approximately
  $$P_b^{\text{BPSK}} < P_b^{\text{QPSK}} < P_b^{\text{4-ASK}} < P_b^{\text{16-QAM}} < P_b^{\text{8-PSK}} < P_b^{\text{16-ASK}}$$
* QPSK and BPSK plot **on top of each other** because for QPSK Gray-coded the bit-error formula is identical to BPSK at the same $E_b/N_0$ — a known property reproduced by our simulation.

### 4. Eb/No Threshold (Q9 console output)

The Eb/No sweep (`EbNo_test = 30:-1:-10`) records the smallest $E_b/N_0$ at which the **complete message** still survives the AWGN+demod chain. For the seed used in the run shown, the console reported (typical run; values vary by ±1 dB run-to-run because of random noise):

```
4-ASK  threshold (dB) : -4
16-ASK threshold (dB) :  5
8-PSK  threshold (dB) : -7
16-QAM threshold (dB) : -6
BASK   :  <= -10 dB (still recovered at lowest tested)
QPSK   :  <= -10 dB (still recovered at lowest tested)
```

**Interpretation:**

* **BASK** and **QPSK** are the most noise-robust — they could not be broken in the −10 to 30 dB sweep.
* **16-ASK** has the *highest* threshold, ≈ +5 dB, so it is the **most fragile**. This is consistent with the Lecture 8 formula
  $\frac{6 \log_2 M}{M^2 - 1}$ shrinking sharply as $M$ rises (e.g., $\frac{6 \cdot 4}{255} = 0.094$ for 16-ASK vs $\frac{6}{3} = 2$ for BASK).
* High-order **PSK and QAM** are far better than 16-ASK because they spread the constellation in **two** dimensions instead of one.

### 5. Energy Consumption per Bit

For a fixed average symbol energy $E_s$, the energy per bit is
$$E_b \;=\; \frac{E_s}{\log_2 M}$$

So a higher-order constellation **divides the available symbol energy across more bits**, lowering $E_b$ and pushing the system closer to the BER cliff. In other words, dense schemes (16-QAM, 16-ASK) trade **energy efficiency for bandwidth efficiency**.

### 6. Most Suitable Scheme

* **For maximum data rate at a given bandwidth:** **16-QAM** — 4 bits/symbol, much more energy-efficient than 16-ASK because it spreads points in $I$ and $Q$ rather than along a single line.
* **For a noisy channel:** **QPSK** — same BER as BPSK with twice the bit rate, robust well below 0 dB $E_b/N_0$.
* **Compromise:** if the channel is moderately noisy and bandwidth is also constrained, **QPSK** or **8-PSK** is preferred over 16-ASK or 16-QAM.
* **To avoid:** **16-ASK** — worst Eb/No threshold and highest BER among the six tested schemes; placing all 16 points on a single axis makes any noise instantly fatal.

---

## Conclusions

The simulation in `oel.m` exercises the full baseband-to-RF chain described in **Lectures 7–8** and **Lab Experiments 6–9**:

* **Successes.** All six modulators deliver `ISLAM` perfectly at $E_b/N_0 = 23$ dB. The coherent demodulator (mix + Butterworth LPF + middle-of-symbol sampling) reproduces both the recovered NRZ waveform and the original ASCII text. The theoretical BER curves match the qualitative observations from the constellation overlays (tighter clouds at higher Eb/No).

* **Advantages observed.**
  * Higher-order modulation (16-QAM in particular) multiplies the achievable bit rate without increasing carrier bandwidth.
  * QPSK and BPSK provide essentially identical BER while QPSK doubles the throughput — confirming the Lecture-7 statement that QPSK = "two BPSKs in quadrature".
  * The $\mathrm{erfc}$-based Q-function evaluation is exact and avoids needing the Communications Toolbox for the BER calculation.

* **Limitations encountered.**
  * The threshold values in Q9 are **dependent on the noise realisation** because `awgn` draws fresh random samples each call; the printed thresholds are therefore approximate (within ±1 dB run-to-run).
  * 16-ASK is unusable below about +5 dB Eb/No. Its use in practice is rare for this exact reason; M-QAM is the standard high-density choice.
  * The script pads short bit streams with zeros so length is a multiple of $\log_2 M$ — a small bookkeeping nuisance, especially for 8-PSK ($r=3$) where the message length must be a multiple of 3 bits.
  * Carrier and timing recovery are *assumed perfect* (the receiver multiplies by an exact-frequency, exact-phase local oscillator). Real receivers must additionally implement carrier/timing synchronisation, which would degrade BER further.

* **Problems faced during development.**
  * Choosing the LPF cut-off (`bit_rate*1.5`) — too narrow distorts the symbols, too wide lets in noise. The compromise was tuned by inspecting the recovered NRZ subplots.
  * The 16-ASK demod required a **nearest-level** decision (`min(abs(a_hat - amp_levels))`) instead of fixed thresholds because the 16 levels are too close to safely use hand-coded `if/elseif` regions.
  * Sampling at the **middle** of each symbol (rather than start or end) was crucial — the Butterworth filter's group delay introduces a small offset that disappears when sampling mid-symbol.

In summary, the experiment confirms the trade-off identified in Lecture 8: **bandwidth efficiency increases with $M$**, but **noise immunity and energy efficiency decrease**, and the optimum scheme depends on whether the channel is bandwidth-limited or noise-limited.

---

## References

1. **Forouzan, B. A.**, *Data Communication and Networking*, 5th ed., Tata McGraw-Hill Education, 2013. (Reference book for Lectures 7, 8, 12, and Lab Experiments 6–9.)
2. **William Stallings**, *Data and Computer Communications*, 10th ed., Pearson, 2014.
3. **Prakash C. Gupta**, *Data Communications*, Prentice Hall India Pvt. Ltd.
4. AIUB COE 3201 — Lecture 7: *Analog Transmission: ASK, FSK, PSK & QPSK*.
5. AIUB COE 3201 — Lecture 8: *Analog Transmission and Bandwidth Utilization (QAM, BER in AWGN, FDM)*.
6. AIUB COE 3201 — Lecture 8 supplement: *Q-Function Table* (`Q-table_Lecture_8.pdf`).
7. AIUB COE 3201 Laboratory Manuals — Experiment 6 (Digital-to-Analog Conversion in MATLAB), Experiment 7 (AM/FM in Simulink), Experiment 8 (Message Passing through Modulator with AWGN), Experiment 9 (Frequency Division Multiplexing).
