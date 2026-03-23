# MethodUsed: Volume per Atom Predictor

## 1. Problem Definition

The objective of this method is to predict the **volume per atom (VPA)** of inorganic crystalline compounds directly from **composition-only information**, augmented with a small set of **physics-informed elemental descriptors**. The model is designed to output not only a point estimate (μ) but also a calibrated uncertainty (σ), enabling predictions in the form:

> **VPA = μ ± σ (Å³/atom)**

This formulation is particularly suitable for **downstream crystal structure generation**, where VPA acts as a density prior independent of symmetry or lattice choice.


## 2. Motivation for Predicting Volume per Atom

Predicting **total unit-cell volume** directly introduces ambiguity because the total volume depends on:

* Choice of conventional vs primitive cell
* Number of atoms per unit cell (Z)
* Symmetry-dependent representations

In contrast, **volume per atom** is:

* Largely invariant across polymorphs
* Dominated by atomic size and bonding chemistry
* A physically meaningful, transferable quantity

Therefore, volume per atom is chosen as the primary regression target.


## 3. Dataset Construction

### 3.1 Data Source

All data is obtained from the **Materials Project (MP)** database using the modern `mp-api`. The following raw information is retrieved for each material:

* Material ID
* Chemical composition (formula)
* Crystal structure

No symmetry, lattice parameters, or Wyckoff information is used as model input.


### 3.2 Correct Computation of Volume per Atom

For each MP entry:

* The **total unit-cell volume** is taken from the stored crystal structure
* The **number of atoms** is computed as the number of atomic sites in the structure

[ V_{atom} = \frac{V_{cell}}{N_{atoms}} ]

**Important correction:**
The number of atoms is **not** derived from the reduced chemical formula, as MP structures are often stored in conventional cells. Using the reduced formula would introduce systematic label errors, especially for polymorphs.


### 3.3 Polymorph Handling and Deduplication

Multiple MP entries may correspond to the **same chemical composition** but differ in:

* Space group
* Conventional cell choice
* Symmetry setting

Since volume per atom is expected to be nearly invariant across polymorphs, all entries sharing the same composition are **grouped**, and their VPA values are **aggregated using the mean**.

This step:

* Removes label noise
* Prevents over-representation of common elements
* Improves regression stability


## 4. Feature Representation

The final dataset contains the following columns:

```
composition
elements
fractions
volume_per_atom
mean_atomic_radius
std_atomic_radius
mean_electronegativity
mean_valence_electrons
mean_atomic_mass
```

### 4.1 Composition Encoding

Each compound is represented by:

* A list of atomic numbers (`elements`)
* Corresponding normalized stoichiometric fractions (`fractions`)

This allows the model to operate on variable-length compositions.


### 4.2 Physics-Informed Elemental Features

To explicitly encode atomic size and bonding physics, the following **composition-averaged elemental descriptors** are computed using `pymatgen`:

| Feature                | Physical Meaning                               |
| ---------------------- | ---------------------------------------------- |
| Mean atomic radius     | Controls atomic packing and volume             |
| Std atomic radius      | Measures size mismatch and packing frustration |
| Mean electronegativity | Proxy for bond strength                        |
| Mean valence electrons | Proxy for bond density                         |
| Mean atomic mass       | Correlates with atomic size and density        |

These features are computed as **stoichiometrically weighted averages**.

This approach avoids high-dimensional handcrafted descriptors (e.g., full Magpie) while retaining physical interpretability.


## 5. Dataset Splitting

The dataset is randomly split into:

* 80% training
* 10% validation
* 10% test

Splits are performed **after composition-level deduplication** to avoid information leakage.


## 6. Model Architecture

The model consists of three main components:

### 6.1 Composition Encoder

* Element embeddings (trainable, dimension = 64)
* Stoichiometric weighted pooling
* Two-layer MLP

This branch learns periodic trends and element–element interactions.


### 6.2 Physics Feature Encoder

A separate MLP processes the five scalar physics features. This ensures:

* Explicit use of size and bonding information
* Clean separation between learned and handcrafted features


### 6.3 Fusion and Output Heads

The outputs of the two encoders are concatenated and passed through a fusion network, followed by two heads:

* Mean head (μ): predicts volume per atom
* Log-variance head (log σ²): predicts uncertainty


## 7. Loss Function (Key Novelty)

The model is trained using **Gaussian Negative Log Likelihood (NLL)**:

[ L = \frac{(y - \mu)^2}{2\sigma^2} + \frac{1}{2} \log \sigma^2 ]

This loss:

* Penalizes overconfident predictions
* Encourages calibrated uncertainty
* Enables direct μ ± σ reporting


## 8. Training Procedure

* Optimizer: Adam
* Learning rate: 1e-3
* Batch size: 256
* Epochs: 150–200

The model with the **lowest validation MAE** is saved.


## 9. Evaluation Metrics

The following metrics are reported:

### 9.1 Accuracy

* Mean Absolute Error (MAE) on test set

### 9.2 Uncertainty Quality

* Pearson correlation between predicted σ and absolute error

### 9.3 Calibration

* Coverage within μ ± σ

### 9.4 Risk-Controlled Screening

* Coverage vs MAE curves by thresholding σ


## 10. Experimental Observations

* Increasing epochs improves MAE but may slightly degrade uncertainty–error correlation
* Early stopping yields more informative uncertainty
* Physics features dramatically reduce MAE compared to composition-only models

Typical final performance:

* Test MAE ≈ 0.64–0.68 Å³/atom
* Uncertainty–error correlation ≈ 0.52–0.68


## 11. Suitability for Crystal Structure Generation

The final model is well-suited for downstream structure generation because:

* It is composition-only
* It avoids symmetry leakage
* It provides calibrated density priors

Predicted volume per atom can be combined with independently predicted space group and atom count to reconstruct total cell volume.


## 12. Summary

This method presents a robust, physics-informed, uncertainty-aware approach for predicting volume per atom from composition. By correcting dataset construction errors, handling polymorphs appropriately, and incorporating minimal yet meaningful elemental descriptors, the model achieves strong accuracy while remaining interpretable and extensible.
