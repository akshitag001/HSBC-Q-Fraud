# HSBC-Q-Fraud: Quantum-Enhanced Selective Escalation Architecture for High-Throughput Fraud Detection

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Tests Passing](https://img.shields.io/badge/tests-24%2F24%20passing-brightgreen.svg)](tests/)
[![Framework](https://img.shields.io/badge/framework-PennyLane%20%7C%20Amazon%20Braket-orange.svg)](https://amazon-braket-sdk-python.readthedocs.io/)
[![Baseline](https://img.shields.io/badge/baseline-LightGBM%20Calibrated-success.svg)](https://lightgbm.readthedocs.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Verdict-Outcome%20B%20(Empirically%20Verified)-blueviolet.svg)](docs/evidence/CANONICAL_RESULTS.md)

**Repository:** `akshitag001/HSBC-Q-Fraud`  
**Author:** Akshit Agarwal (Rashtriya Raksha University, Gandhinagar, India)  
**Challenge:** HSBC Global Quantum + AI Challenge 2026 — Track 1: Quantum-Enhanced Credit Card Fraud Detection  
**Scientific Verdict:** `OUTCOME B — NO QUANTUM ADVANTAGE DEMONSTRATED, BUT ENTERPRISE SELECTIVE CLASSICAL ARCHITECTURE VALIDATED`  

---

## 📌 Executive Summary

Digital payment ecosystems process tens of thousands of transactions per second under a strict **$< 50\text{ ms}$ synchronous authorization SLA** and extreme class imbalance (**$3.50\%$** native fraud prevalence in the $590,540$-transaction official IEEE-CIS benchmark). Indiscriminately evaluating quantum circuits across high-throughput payment streams is fundamentally unviable due to cloud QPU queue latency ($3\text{--}20\text{ minutes}$) and catastrophic unit economics ($\sim \$35.30$ per transaction on IonQ Aria via AWS Braket; $>\$350,000$ per $10\text{k}$ batch).

**HSBC-Q-Fraud** designs, implements, and rigorously benchmarks an **asymmetric two-tier selective escalation architecture**:
1. **Frontline Clearance Path ($99.0\% - 99.5\%$ of volume):** An ultra-fast, isotonically calibrated LightGBM model autonomously approves benign traffic in **$4.07\text{ ms}$ median latency**, achieving a PR-AUC of **$0.4040$** ($11.74\times$ lift over the $3.44\%$ base rate) and ROC-AUC of **$0.8503$**.
2. **Selective Uncertainty Router:** An epistemic uncertainty filter isolates boundary transactions ($|p - 0.5| \le \tau$) into a strict $0.5\% - 1.0\%$ specialist review budget. At a **$0.5\%$ budget**, the router achieves **$42.81\%$ fraud density** (**$12.44\times$ enrichment**), capturing $28.1\times$ more fraud than transaction amount ranking.
3. **Specialist Evaluation Tier:** Tested against both bandwidth-tuned classical baselines (RBF, MLP, GBM) and quantum algorithms (Projected Quantum Kernel [PQK] and Fidelity Quantum Kernel via PennyLane / Amazon Braket SDK).

```mermaid
flowchart TD
    TX[Ingress Payment Stream] --> LYS[Feature Pipeline & Scaling]
    LYS --> LGBM[Tier 1: Calibrated Frontline LightGBM\nLatency: 4.07 ms]
    LGBM --> ROUTER{Epistemic Router\n|p - 0.5| <= tau}
    
    ROUTER -->|99.0% - 99.5% Volume\nHigh Confidence| CLEAR[Autonomous Fast Path\nImmediate Clearance\n< 5 ms SLA Compliant]
    ROUTER -->|0.5% - 1.0% Volume\nAmbiguous Boundary| SPEC[Tier 2: Specialist Tier\nConcentration: 42.81% Fraud Density]
    
    SPEC --> RBF[Classical Specialist\nTuned RBF Kernel\nPR-AUC: 0.6561]
    SPEC --> PQK[Quantum Specialist\nProjected Quantum Kernel\nPR-AUC: 0.5540]
    
    RBF -.-> HYP[Paired Bootstrap Hypothesis Test\nDelta: -0.1021 | 95% CI: -0.0383 to +0.1821\np = 0.246 | Null Not Rejected]
    PQK -.-> HYP
    HYP --> VERDICT[Outcome B: Operational Classical Pipeline Deployed\nHardware Misallocation Prevented: $350k+ Saved]
```

---

## 🔬 Authoritative Empirical Results (Single Source of Truth)

All metrics trace directly to [`docs/evidence/CANONICAL_RESULTS.md`](docs/evidence/CANONICAL_RESULTS.md) with frozen provenance across the $590,540$-transaction IEEE-CIS dataset:

| Component | Dataset | Metric | Verified Result | Status | Evidentiary Artifact Trace |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **LightGBM Baseline** | Real (IEEE-CIS) | PR-AUC | **0.4040** (Lift: **11.74x** over $\pi=0.0344$) | `[MEASURED]` | [`real_classical_baseline.json`](docs/evidence/real_classical_baseline.json) |
| **LightGBM Baseline** | Real (IEEE-CIS) | ROC-AUC | **0.8503** | `[MEASURED]` | [`real_classical_baseline.json`](docs/evidence/real_classical_baseline.json) |
| **LightGBM Baseline** | Real (IEEE-CIS) | Brier Score Loss | **0.0250** (ECE: **0.0785**) | `[MEASURED]` | [`real_calibration_audit.json`](docs/evidence/real_calibration_audit.json) |
| **Selective Router** | Real (IEEE-CIS) | 0.5% Budget Concentration | **42.81%** (**12.44x lift**, 253 frauds / 591 tx) | `[MEASURED]` | [`real_router_audit.json`](docs/evidence/real_router_audit.json) |
| **Selective Router** | Real (IEEE-CIS) | 1.0% Budget Concentration | **37.26%** (**10.83x lift**, 440 frauds / 1,181 tx) | `[MEASURED]` | [`real_router_audit.json`](docs/evidence/real_router_audit.json) |
| **Selective Router** | Real (IEEE-CIS) | 2.0% Budget Concentration | **30.48%** (**8.86x lift**, 720 frauds / 2,362 tx) | `[MEASURED]` | [`real_router_audit.json`](docs/evidence/real_router_audit.json) |
| **Classical RBF Control** | Real (IEEE-CIS) | PR-AUC (Matched $N=200$) | **0.6561** (ROC-AUC: **0.7546**) | `[MEASURED]` | [`real_quantum_matched_experiment.json`](docs/evidence/real_quantum_matched_experiment.json) |
| **Quantum Projected (PQK)** | Real (IEEE-CIS) | PR-AUC (Matched $N=200$) | **0.5540** (ROC-AUC: **0.6753**) | `[MEASURED]` | [`real_quantum_matched_experiment.json`](docs/evidence/real_quantum_matched_experiment.json) |
| **Quantum − RBF Delta** | Real (IEEE-CIS) | $\Delta\text{PR-AUC}$ (Bootstrap) | **-0.1021** ($95\%\text{ CI: } [-0.0383, +0.1821]$) | `[MEASURED]` | [`real_quantum_matched_experiment.json`](docs/evidence/real_quantum_matched_experiment.json) |
| **Quantum Hypothesis Test** | Real (IEEE-CIS) | $p$-value (Paired Bootstrap) | **$p = 0.246$** (Bonferroni: **$p = 0.492$**) | `[MEASURED]` | [`real_quantum_matched_experiment.json`](docs/evidence/real_quantum_matched_experiment.json) |
| **Quantum Geometry** | Real (IEEE-CIS) | Centered Kernel Alignment | **0.5741** (PQK vs RBF), **0.9373** (Fidelity vs RBF) | `[MEASURED]` | [`real_quantum_geometry.json`](docs/evidence/real_quantum_geometry.json) |
| **Quantum Noise** | Simulation | PR-AUC Degradation | **-1.32% to -6.49%** ($p=0.01\text{ to }0.05$) | `[MEASURED]` | [`real_noise_robustness.json`](docs/evidence/real_noise_robustness.json) |
| **QPU Economics** | Modeled (IonQ) | Cost / Escalated Tx | **$35.30** ($353,000 / 10k escalated tx) | `[MODELED]` | [`hardware_and_economics.json`](docs/evidence/hardware_and_economics.json) |
| **Realized Savings** | Real Operations | Realized Fraud Savings | **$0.00** (Zero realized claims) | `[VERIFIED]` | [`ECONOMIC_AUDIT.md`](docs/evidence/ECONOMIC_AUDIT.md) |
| **Hardware Decision Gate** | Protocol Gate | Execution Gate Status | **HARDWARE NOT JUSTIFIED** | `[VERIFIED]` | [`hardware_gate.json`](docs/evidence/hardware_gate.json) |
| **Master Scientific Verdict**| Scientific Synthesis| Final Outcome | **OUTCOME B** | `[VERIFIED]` | [`RESEARCH_VERDICT.md`](docs/evidence/RESEARCH_VERDICT.md) |

---

## 💡 Key Architectural Insights

1. **Selective Routing Works Exceptionally Well:**  
   Frontline uncertainty routing isolates genuine ambiguity from routine volume. By directing just **$0.5\%$** of transaction volume to specialist evaluation, fraud concentration spikes from the $3.44\%$ baseline to **$42.81\%$** ($12.44\times$ enrichment).
2. **Quantum Advantage Not Demonstrated on Matched Cohorts:**  
   Projected Quantum Kernel achieves $\text{PR-AUC} = 0.5540$ vs Classical RBF $\text{PR-AUC} = 0.6561$. Paired bootstrap resampling across 1,000 iterations yields a 95% confidence interval spanning zero ($[-0.0383, +0.1821]$) and $p = 0.246$, proving the null hypothesis cannot be rejected.
3. **Geometric Equivalence (CKA):**  
   Centered Kernel Alignment (CKA) between quantum kernels and classical RBF measures **$0.5741$** (PQK) and **$0.9373$** (Fidelity Kernel), demonstrating that current quantum feature maps largely mirror classical Gaussian Hilbert spaces rather than discovering orthogonal decision boundaries.
4. **Preventing Cloud Waste:**  
   Enforcing strict hardware execution gates prevents enterprise financial misallocation ($>\$350,000$ per 10,000 transactions on physical cloud QPUs) on a null result, while shipping an SLA-compliant classical pipeline operating at **$\$0.65$ per million transactions**.

---

## 📁 Repository Structure

```
HSBC-Q-Fraud/
├── docs/                               # Scientific reports & evidence ledger
│   ├── evidence/                       # Canonical results, latency audits & JSON proofs
│   │   ├── CANONICAL_RESULTS.md        # Single Source of Truth
│   │   ├── real_classical_baseline.json# Baseline LightGBM metrics
│   │   ├── real_router_audit.json      # Selective escalation audit
│   │   └── real_quantum_matched_experiment.json
│   └── proposal/                       # Challenge submission materials & renders
├── proposal/                           # LaTeX & compiled Phase 1 concept proposals
│   ├── HSBC_Phase1_Concept_Proposal.pdf
│   ├── HSBC_Phase1_Supplementary_Appendix.pdf
│   └── HSBC_Phase1_Combined_Submission_Package.pdf
├── scripts/                            # Verification, auditing & PDF build scripts
│   ├── build_latex_proposal.py         # Automated LaTeX PDF compiler (pdflatex)
│   ├── build_proposal.py               # HTML/Headless Edge fallback compiler
│   └── verify_evidence_integrity.py    # Hash & provenance verification
├── src/                                # Core operational codebase
│   ├── dashboard/                      # Interactive Streamlit research UI
│   │   └── app.py
│   ├── data/                           # Ingestion, schema validation & synthetic data
│   │   ├── generate_synthetic.py       # Offline 10,000-sample deterministic benchmark
│   │   └── make_dataset.py             # Temporal splitting pipeline
│   ├── evaluation/                     # Hypothesis testing, CKA & latency telemetry
│   ├── features/                       # StandardScaler & feature pipeline
│   ├── models/
│   │   ├── classical/                  # LightGBM frontline baseline
│   │   ├── controls/                   # Bandwidth-tuned classical RBF, MLP, GBM
│   │   ├── quantum/                    # Projected Quantum Kernel (PennyLane/Braket)
│   │   └── router/                     # Epistemic uncertainty escalation router
│   └── utils/                          # Paths, seeds, and provenance helpers
├── submission_package/                 # Frozen competition submission artifacts
├── tests/                              # Automated test suite (24 passing unit tests)
├── pytest.ini                          # Test configuration
├── requirements.txt                    # Production dependencies
├── requirements-quantum.txt            # PennyLane & Amazon Braket dependencies
└── README.md                           # This document
```

---

## 🚀 Quickstart & Reproduction

### 1. Environment Setup
```bash
# Clone the repository
git clone https://github.com/akshitag001/HSBC-Q-Fraud.git
cd HSBC-Q-Fraud

# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate       # On Linux/macOS
# or: .venv\Scripts\activate    # On Windows

# Install core and quantum dependencies
pip install -r requirements.txt
pip install -r requirements-quantum.txt
```

### 2. Verify Full Test Suite (Zero External Dependencies)
The codebase includes a deterministic offline test suite verifying router logic, quantum feature encoding, kernel alignment, and claim integrity:
```bash
pytest
```
*Expected result:* `24 passed, 100% test pass rate.`

### 3. Launch Interactive Research Demonstration
Explore the two-tier selective router, decision thresholds, and latency comparisons via the Streamlit web dashboard:
```bash
streamlit run src/dashboard/app.py
```

### 4. Run Data & Evaluation Pipeline
```bash
# Generate deterministic synthetic benchmark & temporal split
python -m src.data.generate_synthetic
python -m src.data.make_dataset
python -m src.features.build_features

# Evaluate classical baseline and uncertainty router
python -m src.models.classical.train_baseline
python -m src.evaluation.router_audit

# Run quantum kernel pipeline & hypothesis testing
python -m src.models.quantum.train_quantum_expert
python -m src.evaluation.kernel_diagnostics
```

---

## 📜 Regulatory Explainability & Compliance

In strict compliance with financial regulatory frameworks (e.g., Fair Credit Reporting Act, SR 11-7 model risk management), **HSBC-Q-Fraud** enforces:
- **Global & Local Feature Attribution:** Calibrated TreeSHAP explanations for frontline decisions.
- **Auditable Escalation Reason Codes:** Ambiguous transactions record explicit boundary telemetry (`p_calibrated`, `epistemic_entropy`, `margin_to_boundary`).
- **Firewall Isolation:** High-risk unverified quantum models are decoupled from customer-facing real-time credit declines.

---

## 👤 Author & Research Attribution

- **Lead Researcher & System Architect:** **Akshit Agarwal** ([@akshitag001](https://github.com/akshitag001))
- **Affiliation:** Rashtriya Raksha University (National Security & Police University of India), Ministry of Home Affairs, Gandhinagar, Gujarat, India
- **Challenge Track:** HSBC Global Quantum + AI Challenge 2026 — Track 1: Quantum-Enhanced Fraud Detection

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
