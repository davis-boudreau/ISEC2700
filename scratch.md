## Step 4 – Risk Analysis (Redesigned)

Risk analysis evaluates **likelihood × impact** using a structured scoring model aligned with **FAIR** principles (frequency, vulnerability, and loss magnitude).

# **Scoring Criteria (3×3 model)**

**Likelihood (L)**

| Score          | Description                                 | Weight |
| -------------- | ------------------------------------------- | ------ |
| **1 – Low**    | Rare, strong controls present               | 0.3    |
| **2 – Medium** | Possible, controls partially effective      | 0.6    |
| **3 – High**   | Expected to occur, weak or missing controls | 1.0    |

**Impact (I)**

| Score          | Description                                      | Weight |
| -------------- | ------------------------------------------------ | ------ |
| **1 – Low**    | Minimal disruption, negligible cost              | 0.3    |
| **2 – Medium** | Noticeable operational or financial loss         | 0.6    |
| **3 – High**   | Severe damage, major cost or regulatory exposure | 1.0    |

# **FAIR‑Aligned Quant Model (simplified)**

    Risk = (Threat Event Frequency × Vulnerability) × Loss Magnitude

Mapped to the matrix:

**Likelihood = Threat Event Frequency × Vulnerability**
**Impact = Loss Magnitude (direct + indirect)**

# Mermaid (FAIR summary)



# ## Step 5 – Risk Assessment (3×3 Matrix)

The following **3×3 risk matrix** determines priority levels.

# **3×3 Risk Matrix Categories**

| Impact ↓ / Likelihood → | 1 (Low) | 2 (Medium) | 3 (High)     |
| ----------------------- | ------- | ---------- | ------------ |
| **3 – High**            | Medium  | High       | **Critical** |
| **2 – Medium**          | Low     | Medium     | High         |
| **1 – Low**             | Low     | Low        | Medium       |

# Mermaid (3×3 Matrix)

# Example Scored Risks (Using the new model)

| Risk            | L | I | Rating       |
| --------------- | - | - | ------------ |
| No MFA          | 3 | 3 | **Critical** |
| No Segmentation | 3 | 2 | High         |
| Weak Passwords  | 2 | 2 | Medium       |

# Interpretation

**Critical** → must be addressed immediately\
**High** → prioritized remediation\
**Medium** → planned reduction\
**Low** → acceptance or monitoring

***

If you want, I can also generate:

* a **5×5 matrix**,
* weighted heatmaps,
* FAIR Monte‑Carlo simulation outputs,
* or a full risk register template.
