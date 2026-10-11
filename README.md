# 6G ISAC Slicing Performance analysis (FR2/mmWave)
>  Performance analysis of ISAC Slicing with Sensing-Assisted Communication and Communication-Assisted Sensing


## 1. Introduction to Interdependent 6G ISAC Slicing
In **6G networks**, **Integrated Sensing and Communication (ISAC)** transforms mobile infrastructure from a simple data pipe into a spatially aware platform. 
While traditional network slicing treats **Sensing-as-a-Service (SaaS)** and **Communication-as-a-Service (CaaS)** as distinct, isolated slices competing for hardware and spectrum, modern 6G paradigms utilize an interdependent slicing framework. 

Rather than working against each other, the two slices create a reciprocal ecosystem driven by **Sensing-Assisted Communication (SAC)** and **Communication-Assisted Sensing (CAS)**.

---

## 2. The Two Pillars of Interdependent ISAC Slicing

Instead of traditional, hard-partitioned resource allocation, 6G ISAC slicing optimizes the network by exploiting the cross-slice synergies between SAC and CAS.

```layout

                  ┌──────────────────────────────┐
                  │  6G ISAC Slicing Framework   │
                  └──────────────┬───────────────┘
                                 │
         ┌───────────────────────┴───────────────────────────────┐
         |                                                       |
         ▼                                                       ▼
┌───────────────────────────────-──┐             ┌─────────────────────────────────-┐
│  Sensing-Assisted Comm. (SAC)    │             │ Communication-Assisted Sen. (CAS)│
├─────────────────────────────────-┤             ├─────────────────────────────────-┤
│ • Objective: Enhance Data Links  │             │ • Objective: Enhance Precision   │
│ • Uses radar data for beamforming│             │ • Passes sensing data over       │
│    & handover/Mobility.          │             │   communication channels         |
|                                  |             |   for fusion.                    │
└─────────────────────────────────-┘             └────────────────────────────────-─┘

```

### 2.1 Sensing-Assisted Communication (SAC)
In the SAC paradigm, the **sensing slice gathers real-time physical environmental context to optimize data links**.
* **Mechanics:** The network uses radar-like RF reflections to track physical target positions, user equipment (UE) movements, and environmental obstacles in real time.
* **Slicing Synergies:** By passing this spatial context to the communication slice, the network can perform ultra-fast beamforming, instant beam-failure recovery, and predictive handovers without relying on heavy pilot signal overhead. This significantly reduces latency and increases the throughput of the communication slice.

### 2.2 Communication-Assisted Sensing (CAS)
In the CAS paradigm, the **communication slice serves as the data transport and coordination mechanism to enhance sensing capability**.
* **Mechanics:** High-resolution sensing typically requires bi-static or multi-static configurations, where multiple distributed base stations and UEs coordinate to sense a target.
* **Slicing Synergies:** The communication slice provides the ultra-low latency, high-bandwidth links required for these distributed sensors to exchange raw RF/IQ data, coordinate flash synchronization, and execute sensor fusion algorithms. Without the communication slice routing this data, the sensing slice would be limited to low-resolution, short-range monostatic configurations.

__NOTE__: Ultimately, successful 6G ISAC slicing relies on Dynamic Resource Allocation frameworks (often managed via AI/ML at the orchestration layer). Instead of treating spectrum as a fixed pie split between two competing services, the network dynamically tilts resources based on environmental demand allocating wider bands to sensing when target tracking is **critical** to prevent communication dropouts, and shifting it back to data transmission once a stable, sensing-assisted connection is locked in.

---

## 3. Mathematical Optimization and Waveform Architecture

### 3.1 Joint Resource Allocation Framework (!! Under construction !!)
To optimize the shared resource allocation between the communication throughput R and the sensing precision (measured via Cramér-Rao Bound or Mean Squared Error E), 
we need to define a multi-objective utility maximization problem under a strict total power constraint \(P_{\max}\):

```text

\[\max_{\rho} \quad \mathcal{U}(\rho) = \alpha \cdot \bar{R}(\rho, E) + (1-\alpha) \cdot \bar{S}(\rho, R)\]

Where:
* \(\rho \in [0, 1]\) represents the power allocation factor dedicated to communication channels.
* 1-ρ represents the power remaining for radar/sensing emissions.
* α is the prioritization coefficient balancing user traffic demands against tracking resolution.
* R̄ and S̄ represent the normalized communication throughput and sensing precision scores, respectively, factoring in cross-slice alignment bonuses.

```

### 3.2 Waveform Comparison Matrix
Choosing the right physical layer (L1) waveform is critical for managing high-mobility scenarios (e.g., connected autonomous vehicles, drones) without causing severe cross-slice degradation:

| Waveform Type | Communication Efficiency | Sensing Resolution | Doppler/Mobility Resilience | Algorithmic Complexity | Implementation & Processing Complexity |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CP-OFDM** | High spectral efficiency, native 5G compatibility. | Moderate; prone to range-Doppler coupling ambiguities. | Low; suffers from Inter-Carrier Interference (ICI) under high mobility. | Low | Low: Utilizes standard low-complexity FFT/IFFT processing pipelines. |
| **DFT-s-OFDM** | High spectral efficiency, native 5G compatibility. | Moderate; prone to range-Doppler coupling ambiguities. | Low; suffers from Inter-Carrier Interference (ICI) under high mobility. | Low | Low: Utilizes standard low-complexity FFT/IFFT processing pipelines. |
| **OTFS (Delay-Doppler)** | High; converts time-varying channels into time-invariant structures. | Exceptionally high range and velocity resolution. | Full resilience; cleanly maps directly to physical coordinate geometries. | High (Requires Symplectic Finite Fourier Transforms) | High: Requires complex Symplectic Finite Fourier Transforms (SFFT) and advanced equalizers. |



---

## 4. End-to-End Resource Simulation Script (draft) - WIP
The following draft complete, executable Python script simulates the power sharing and performance trade-offs between the two slices, 
incorporating cross-slice alignment bonuses (SAC alignment bonus and CAS fusion gain).

```python
import numpy as np

def simulate_isac_tradeoff(rho, P_max_dbm=46.0, B_hz=100e6, N0_dbm_hz=-174.0):
    """
    Simulates the performance metrics for a 6G ISAC Slicing Framework.
    
    Parameters:
        rho (float): Power allocation factor for communication (0.0 to 1.0).
        P_max_dbm (float): Maximum transmit power in dBm.
        B_hz (float): Total shared system bandwidth in Hz.
        N0_dbm_hz (float): Noise spectral density in dBm/Hz.
    """
    # Convert powers from log scale to linear Watts
    P_total = 10**((P_max_dbm - 30) / 10)
    P_comm = rho * P_total
    P_sense = (1.0 - rho) * P_total
    
    # Path loss and noise floor models
    noise_power = 10**((N0_dbm_hz + 10 * np.log10(B_hz) - 30) / 10)
    h_comm = 1e-4  # Baseline channel attenuation factor
    h_sense = 5e-5 # Baseline radar cross-section reflection coefficient
    
    # Baseline Signal-to-Noise Ratios
    snr_comm_base = (P_comm * h_comm) / noise_power
    snr_sense_base = (P_sense * h_sense) / noise_power if P_sense > 0 else 1e-6
    
    # Cross-Slice Mutual Alignment Dependencies
    # 1. CAS Fusion Gain: Sensing precision benefits from high communication data routing
    cas_gain = 1.0 + 0.5 * np.log10(1.0 + snr_comm_base)
    sensing_precision = np.sqrt(snr_sense_base) * cas_gain if P_sense > 0 else 0.0
    
    # 2. SAC Alignment Bonus: Communication tracking overhead drops with precise physical state info
    sac_bonus = 1.0 + 0.3 * (1.0 - 1.0 / (1.0 + sensing_precision)) if sensing_precision > 0 else 1.0
    comm_throughput = B_hz * np.log2(1.0 + snr_comm_base) * sac_bonus
    
    return comm_throughput / 1e6, sensing_precision

if __name__ == "__main__":
    print("=" * 65)
    print(f"{'Comm Power (Rho)':<18}{'Throughput (Mbps)':<25}{'Sensing Precision':<20}")
    print("=" * 65)
    
    # Loop over the trade-off continuum from sensing-heavy to communication-heavy configurations
    for r in np.linspace(0.1, 0.9, 9):
        throughput_mbps, precision_score = simulate_isac_tradeoff(r)
        print(f"{r:<18.2f}{throughput_mbps:<25.2f}{precision_score:<20.2f}")
    print("=" * 65)
```

## References
1. https://arxiv.org/html/2610.03879v1
2. https://github.com/DanteCampos/2026-ISAC-DJ


__NOTE__: UL/DL CA + ISAC slicing needs to be defined later. 

__*Disclaimer*__: Above informations needs to be refined, as some informations from AI responses may include mistakes.
