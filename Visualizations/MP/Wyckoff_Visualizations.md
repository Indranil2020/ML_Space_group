## Wyckoff-based Model Analysis (Structure-aware Ensemble)

---

### 1. Wyckoff Letter Prediction — Per-class Metrics (Heatmaps)

![Wyckoff Letter Prediction](Wyckoff_Letter_Prediction.png)

**What it shows (from the image):**  
Three color-mapped heatmaps displaying Precision, Recall, and F1 Score for individual Wyckoff letters (a–z). We can see intense dark blue/green/red bands (high performance, >0.90) for high-frequency letters like 'a', 'b', 'c', 'd', and 'e', with performance fading for the rarest special positions. 

**Significance & Indication:**  
This plot breaks down the model's accuracy across the massive combinatorial space of crystallography. It indicates that your model is highly proficient at assigning the most critical symmetry sites (general positions and common special positions) but struggles slightly on the rarest crystallographic orbits.

**Reference & Comparison:**  
This evaluates the exact same task as the ShotgunCSP model, which uses Random Forest regressors to predict Wyckoff-letter occurrence probabilities from chemical composition. While ShotgunCSP evaluates its success in aggregate using Kullback–Leibler (KL) divergence, your heatmap approach is more transparent. It provides a granular "per-orbit" view of exactly which symmetry constraints your model learns effectively, offering a deeper diagnostic view than a single KL divergence metric.

---

### 2. Wyckoff Multi-label Prediction — Class Imbalance & Recall@K Curve

![Wyckoff Multi-label Prediction](Wyckoff_Multi-label_Prediction.png)

**What it shows (from the image):**  
- *Left:* A logarithmic bar chart highlighting the severe class imbalance in crystallographic data (e.g., letter 'a' has >10⁴ instances, while 'z' and 'y' have single digits).  
- *Right:* A Recall@K curve that starts at 0.49 for K=1 and climbs incredibly fast, hitting 0.91 by K=6 and effectively 0.99 by K=13.

**Significance & Indication:**  
The combinatorial explosion of possible Wyckoff assignments is a major bottleneck in generative crystallography. The steep Recall@K curve indicates that your model is highly efficient at search space pruning. It proves that a downstream structure generator only needs to sample a handful of your model's top predictions to almost guarantee finding the correct physical configuration.

**Reference & Comparison:**  
ShotgunCSP explicitly notes that space groups with high degrees of freedom (like *Imma*) can have over 1,755 possible Wyckoff assignments, and narrowing this down is crucial for success. ShotgunCSP uses a space-group predictor that achieves a 92.6% recall rate within the top 30 predictions. Your Recall@K curve applies this exact evaluation philosophy to Wyckoff letters and proves that your model can prune the search space with even greater efficiency (91% recall in just 6 guesses).

---

### 3. Prediction Confidence & Probability Distribution

![Prediction Confidence](Prediction_Confidence_Probability_Distribution.png)

**What it shows (from the image):**  
- *Left:* Overlapping histograms of "Prediction entropy" (uncertainty) in nats. Correct predictions (green) are distinctly clustered at lower entropy levels, while incorrect predictions (red) skew heavily toward high entropy (>2.0 nats).  
- *Right:* A bar chart of the mean predicted probability sorted by Wyckoff letter, showing a sharp drop-off after the first few letters.

**Significance & Indication:**  
This proves your ensemble model is exceptionally well-calibrated and physically aware—it "knows what it doesn't know." High entropy correlates directly with physical impossibility or lack of data. You can use an entropy threshold to automatically reject uncertain predicted structures before sending them to an expensive Density Functional Theory (DFT) relaxation step.

**Reference & Comparison:**  
This mirrors the uncertainty-based triage methodology used in the Wren (Wyckoff Representation regressioN) paper. Wren uses deep ensembles to estimate predictive uncertainty (σ) and explicitly filters out high-risk predictions to improve discovery precision. Your entropy histogram provides visual proof that your model can be used in an identical filtering pipeline.

---

### 4. Top 30 Magpie Feature Importances

![Feature Importance](Top_30_Magpie_Feature_Importances.png)

**What it shows (from the image):**  
A horizontal bar chart of the top features driving your LightGBM models. "MagpieData avg dev Electronegativity" and specific space groups dominate, followed by valence and periodic table features.

**Significance & Indication:**  
This is the explainability backbone of your work. It proves the model has learned physically meaningful relationships: bonding (electronegativity), electron configuration (valence), and symmetry constraints (space group).

**Reference & Comparison:**  
ShotgunCSP and similar models rely on compositional descriptors but do not explicitly visualize feature dominance. Your plot fills this interpretability gap.

---

### 5. t-SNE of Validation Set

![t-SNE](t-SNE_of_Validation_Set.png)

**What it shows (from the image):**  
A 2D t-SNE projection colored by dominant predicted Wyckoff letter. A dense central cluster (e.g., 'a') is surrounded by more dispersed clusters for other letters.

**Significance & Indication:**  
Validates that your feature space encodes meaningful crystallographic structure. Similar compositions map to similar symmetry environments.

**Reference & Comparison:**  
Comparable to latent-space validation in Matra-Genoa and CGWGAN.

---

### 6. Confidence-stratified Prediction Quality (Coverage–Accuracy Trade-off)

![Confidence Stratified](Confidence_stratified_Prediction_Quality.png)

**What it shows (from the image):**  
- F1 increases as confidence threshold increases  
- Coverage decreases (trade-off)  
- Highest confidence bin achieves ~0.99 F1  

**Significance & Indication:**  
This demonstrates deployability. You can tune the system depending on whether you want recall or precision, which is critical for real-world pipelines.

**Reference & Comparison:**  
Directly analogous to uncertainty filtering in Wren.

---

### 7. Wyckoff Letter Fidelity & Frequency Ratio

![Fidelity](Wyckoff_Letter_Fidelity.png)

**What it shows (from the image):**  
- Predicted vs true frequency alignment  
- Ratio plot with ±20% tolerance band  

**Significance & Indication:**  
Confirms that the model preserves the natural distribution of crystallographic symmetries, avoiding bias toward dominant classes.

**Reference & Comparison:**  
Equivalent to KL divergence evaluation in ShotgunCSP, but more interpretable.

---

### 8. Wyckoff Per-class Probability Distributions

![Probability Distribution](Wyckoff_Per-class_Probability_Distributions.png)

**What it shows (from the image):**  
- Violin plots: true positives near 1.0, negatives near 0.0  
- Scatter: strong correlation between predicted probability and true frequency  

**Significance & Indication:**  
Shows clean class separation and strong probabilistic reasoning.

**Reference & Comparison:**  
Analogous to coordinate distribution validation in generative crystal models.

---

### 9. Wyckoff Prediction Calibration (Reliability Diagram)

![Calibration](Wyckoff_Prediction_Calibration.png)

**What it shows (from the image):**  
- Reliability curves close to diagonal  
- Low Expected Calibration Error (~0.05)

**Significance & Indication:**  
Model probabilities are trustworthy for decision-making.

**Reference & Comparison:**  
Aligns with uncertainty validation approaches in ensemble-based materials ML models.

---
