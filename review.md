# REVIEW


---

## 0. OVERALL — THIS READS LIKE LLM-GENERATED TEXT

The whole paper sounds like it was written by LLM. Here is how I can tell:

- Too many filler phrases like "it is ensured that", "in order to", "has been developed to"
- Every paragraph follows the same pattern: claim, then "because of this", then "therefore"
- Sounds confident but says nothing deep
- Words like "comprehensive", "scalable", "modular and interpretable" are repeated everywhere
- The Discussion (Sec. V.G) just says the same things as the Abstract again

**You must rewrite the paper in your own words with real technical depth.**

---

## 1. TITLE

> "A Composition-Driven Machine Learning Framework for Crystal Structure Generation via Prediction of Formation Energy, Stability, Volume, Space Group, and Wyckoff Configurations"

**Big problem:** The title says "Crystal Structure Generation" but you never generate any crystal structure in this paper. There is no CIF file. No atomic coordinates. No DFT check on a generated structure. You only predict some properties like space group, volume, and Wyckoff letters. Predicting properties is NOT the same as generating a structure.

Maybe we have done something like this: *"Predicting Thermodynamic and Structural Properties of Crystals from Composition Using Machine Learning"*

**Question:** Show me one crystal structure your framework made. A CIF file? Coordinates? If you cannot, take "Generation" out of the title.

---

## 2. ABSTRACT

> "The challenge of finding stable crystalline materials has been one of the major challenges in computational materials science"

- You wrote "challenge...challenges" in one sentence. Fix the repetition.

---

> "...typically solved with expensive first-principle calculations and crystal structure prediction methods."

- It should be **"first-principles"** (with an s). This is a basic term in our field.
- Also wrong: DFT does not "solve" the problem of finding stable materials. DFT calculates properties for a structure you already have. CSP methods like USPEX or AIRSS are what actually search for stable structures. These are two different things mixed into one sentence.

---

> "...the majority of methods have been either reliant upon known crystal structures for input, or focused only on isolated prediction tasks."

- Which methods? Name them. This is too vague.

---

> "...demonstrated competitive performance across all phases of the analysis"

- **"Competitive" compared to what?** You never compare your results to any published work in this paper. Without a comparison, this word means nothing.

---

> "...good accuracy for space group prediction and reasonable recall for certain Wyckoff patterns"

- "Good" and "reasonable" are vague. Put the actual numbers in the abstract.

---

> "...we describe how to combine the results from the various predictions to propose likely crystal structure candidates"

- Where do you do this? I do not see any actual structure built or tested anywhere in the paper.

---

> "Overall, this modular and interpretable framework provides a scalable approach..."

- LLM filler words. What makes it "interpretable"? You use transformer ensembles and XGBoost — where is the interpretation? What makes it "scalable"? You never test scalability.

---

## 3. INTRODUCTION (Section I)

> "The development of innovative crystalline materials is key for ongoing growth in technology such as energy storage, electronics, catalysis and structural engineering."

- Generic opening. Every materials science paper starts like this. Say something specific about YOUR work.

---

> "One of the foremost challenges in materials science is accurately predicting both the thermodynamic stability and crystal structure of a material based on its chemical makeup alone."

- You need to explain WHY this is hard. The main reason is **polymorphism** — the same chemical formula can form different crystal structures. Carbon makes diamond AND graphite. SrTiO3 can be cubic or tetragonal. **Your paper never talks about polymorphism at all.** This is a serious gap.

---

> "...composition-based mrethod..."

- **Typo:** "mrethod" should be "method". Proofread your manuscript.

---

> "However, many current techniques utilize known crystal structures as inputs... [3], [4], [5], [6]."

- **Wrong:** References [3] and [4] are composition-only models. They do NOT need crystal structures as input. You are citing papers that go against your own claim. Did you actually read these papers?

---

> "Crystal structures can be described on a compact basis in terms of multiple thermodynamic or structural descriptors, including formation energy, energy above hull, lattice volume, space group symmetry, and Wyckoff positions."

- **Wrong physics:** Formation energy and energy above hull are NOT crystal structure descriptors. They are thermodynamic properties. A crystal structure is defined by lattice parameters, space group, and atomic positions. You are mixing up two different things here.

---

> "These descriptors characterize the stability, symmetry, and atomic arrangement of crystalline materials [7], [8], [9], [10]."

- Refs [7]-[10] are about space group prediction, not about stability. Wrong citations.

---

> "Simultaneous prediction of these properties is challenging due to the need to perform both regression and classification tasks..."

- Doing both regression and classification is not hard — any ML engineer can do that. The real challenge is that the same composition can give many different structures (polymorphism). You are talking about the wrong challenge.

---

### Contributions List (p.2):

> "1) ...simultaneous prediction of formation energy, energy above hull, and thermal stability."

- These three things are NOT independent. Energy above hull comes FROM formation energies. Stability is just a yes/no version of energy above hull. You are basically predicting the same thing three times. Did you check how correlated these targets are?

---

> "2) We introduce a physics-informed volume prediction model..."

- Using atomic radius and electronegativity as features is NOT "physics-informed." That term means putting physical laws into your model (like in PINNs). You just used good features. Every Magpie-based model does this.

---

> "5) The prediction models will also provide uncertainty quantification..."

- Why is this in future tense ("will also provide")? The rest is in past tense. Was this done or not?

---

> "6) ...a scalable approach for connecting chemical composition to crystal structure descriptors..."

- This is the same as contribution (1) said in different words. Not a new contribution.

---

## 4. DATASET AND PREPROCESSING (Section II)

### Section II.A:

> "...density of functional theory (DFT) calculated properties"

- **Wrong:** It is "density functional theory", not "density of functional theory." This is embarrassing for a paper about computational materials science.

**Questions:**
- What version of the Materials Project API did you use?
- How many entries were there before filtering?
- These details are needed so others can repeat your work. They are missing.

---

- Later I find out the dataset sizes are: 150,200 (thermodynamic), 101,579 (space group), 210,579 (Wyckoff), 151,816 (volume). **Why are these all different?** You say they come from one master dataset, but the numbers do not match. Never explained.

---

### Section II.B:

> "Filtering of rare Wyckoff patterns... to improve the generalization of the models."

- **Wrong ML thinking.** Removing rare classes does not make your model better. It just makes the problem easier and your numbers look better. If your goal is to discover new materials, those new materials will likely have unusual Wyckoff patterns — exactly the ones you threw away.
- How many classes did you remove? What was the cutoff? Not reported.

---

> "Removal of duplicate or inconsistent entries"

- How did you define "duplicate"? Same formula? Same formula AND same space group? The Materials Project has many polymorphs (same formula, different structures). How did you handle those? This matters a lot for data leakage.

---

### Section II.C:

> "Materials were classified as stable if their energy above hull was below a predefined threshold (typically close to 0 eV)"

- **What is the actual number?** 0 eV? 0.01 eV? 0.1 eV? Each choice gives a very different result. "Typically close to 0" is not good enough for a scientific paper.

---

### Section II.E:

> "A standard split of 80:20 training to testing"

- But in Section V.A you say 80/10/10 (train/validation/test). **Which one is true?** You cannot say both.

---

> "No data leakage between training and testing sets"

- How do you know? Did you check for similar compositions in both sets? A random split can put Fe2O3 in training and Fe3O4 in testing — they share very similar Magpie features. This is a known problem (see Bartel et al.) and you ignore it.

---

## 5. FEATURE ENGINEERING (Section III)

> "Elemental properties considered include: ... Electron affinity, etc."

- Do not write **"etc."** in a scientific paper. List all features or give a complete table.

---

> "...approximately 130-140 compositional descriptors"

- **You should know the exact number.** Is it 130? 132? 138? 140? Later you say 138. Pick one and be consistent. Not knowing your own feature count is a bad sign.

---

> "The set of these descriptors was developed to encapsulate the statistical distribution..."

- This sounds like you invented these features. You did not. These are standard Magpie features from Ward et al. (2016). Give credit.

---

> "Feature scaling... allowing every feature to contribute... by having equal contributions to the model's final output."

- **Wrong.** Scaling does NOT make all features contribute equally. It just puts them on similar scales for training. A useless feature stays useless after scaling.
- Also: tree-based models (Random Forest, XGBoost) do not need scaling. You use these models a lot. Did you scale features for them too? If so, why?
- **Important:** Did you fit the scaler on the whole dataset or just the training set? If the whole dataset, that is data leakage.

---

## 6. METHODOLOGY (Section IV)

### Section IV.A — Equation (1):

> C -> {E_f, E_hull, S, V_atom, SG, W}

- **Physics problem:** This equation says one composition gives one set of outputs. But one composition can give MANY different crystal structures (polymorphism). Carbon gives diamond AND graphite. This equation is wrong.
- Also: you use four separate models, not one. This equation makes it look like one model does everything. Misleading.

---

### Section IV.B — Multi-Task Model:

> Equation (3): z = Sum of f_i * E(e_i)

- This is a simple weighted average of element embeddings. It is like what Roost and CrabNet use, but simpler (no attention). You do not cite these papers here even though [4] is in your reference list.
- **Problem:** No interaction terms between elements. The model cannot tell the difference between compositions where element interactions matter (like perovskites ABO3 that can form many different structures depending on which A and B elements are used).

---

> Equation (6): L = L_FE + L_hull + L_stab

- **No weights on the losses.** These losses have very different scales (MSE vs cross-entropy). Without weights, one loss will take over training and the others will be ignored. This is a basic multi-task learning mistake.

---

### Section IV.B.3 — Six Model Variants:

You list TC-MTL, LTC-MTL, E-TC-MTL, O-LTC-MTL, FB-MTL, UQ-MTL — six models. Why so many? Where is the reasoning for each one?

**Questions:**
- What was your hypothesis for each?
- Where is the ablation study?
- Where is proof that multi-task learning helps? You never show a single-task baseline.

---

### Section IV.C — Volume Model:

- The architecture is complex but **no details are given**. How many layers? What activations? What dimensions? Without these, nobody can reproduce your work.

> Equation (12): Gaussian negative log-likelihood loss

- How do you stop sigma from blowing up? No training details given.
- **Simple baseline missing:** Volume per atom is closely related to atomic radius. Did you try a simple formula like V ~ (4/3)*pi*r^3 using average atomic radius? If your ML model barely beats this, the ML is not adding much.

---

### Section IV.D — Space Group Prediction:

> "138 features"

- Earlier you said "130-140". Now it is 138. Be consistent.

> Table II: Random Forest, XGBoost, MLP, RF Ensemble, RF + Crystal System

- Why these specific models? No reason given.
- **Where is the comparison with published results?** You cite Zhao et al. [7] and Venkatraman & Carvalho [8,10] who report their own accuracies. You must compare against them.
- MLP gets only 38.6%. Very low — suggests bad tuning or a bug.

---

> "Crystal System Constraint: A separate model predicts the crystal system..."

- What model? What is its accuracy? If it gets the crystal system wrong, you throw away the correct space group. **How often does this happen?** Never checked.

---

### Section IV.E — Wyckoff Prediction:

> Table III: LightGBM, XGBoost, Multi-label binary classifiers

- Only tree-based models. No neural networks tried. No comparison with WyckoffDiff [14] or ShotgunCSP [1] that you cite.

---

> Equation (14): P(w_i | C, SG)

- **Big question:** Is SG the true space group or the predicted one? If you use the true space group during testing, your results are too good to be real. If you use the predicted one (77.2% accurate), errors pile up. Which one did you use? The paper does not say.

---

### Section IV.F — "Structure Construction":

> "...atomic coordinates are generated analytically from symmetry operations."

- **This does not work.** Wyckoff positions have free parameters. For example, in space group Pm-3m, Wyckoff position 24k has coordinates (0, y, z) where y and z are unknown numbers. You predict only the Wyckoff LETTER (like "a", "b", "k"), not these numbers. Without the free parameters, you cannot get atomic coordinates.
- To build a crystal structure, you need:
  1. Wyckoff site with its free parameters (x, y, z values)
  2. All lattice parameters (a, b, c, alpha, beta, gamma) — not just volume
- **This is a basic crystallography error. It breaks the main claim of your paper.**

---

### Section IV.G — "Wyckoff Blueprint":

> "Lattice parameters are computed from predicted lattice volume and the crystal symmetry"

- **Wrong for most crystal systems:**
  - Cubic: OK, a = V^(1/3)
  - Tetragonal: Need a AND c. Volume alone is not enough.
  - Orthorhombic: Need a, b, AND c.
  - Monoclinic: Need a, b, c, AND beta.
  - Triclinic: Need a, b, c, alpha, beta, AND gamma.
- Volume is ONE number. Non-cubic crystals need 2 to 6 numbers. **You cannot get multiple numbers from one number. This is basic crystallography and it is wrong here.**

---

## 7. RESULTS AND DISCUSSION (Section V)

### Section V.A — Thermodynamic Properties:

> "150,200 materials... split into training (80%), validation (10%), and test (10%)"

- Section II.E said 80/20 split. Now it is 80/10/10. **These do not match.**

---

**Table IV results:**

- **Formation Energy MAE = 0.0975 eV/atom:** Published composition-only models like Roost get ~0.028-0.05 eV/atom. Your best result is 2-3 times worse. You never mention this.
- **Hull Distance MAE = 0.059-0.075 eV:** Many real metastable materials have E_hull of 0.01-0.05 eV/atom. Your error is bigger than the values you are trying to predict. This means you cannot tell stable from slightly unstable materials.
- **Stability F1 = 0.93:** Sounds OK, but what is the class balance? If 90% of your data is one class, just guessing that class gives F1 ~ 0.95. Report precision, recall, and how many materials are in each class.
- **No comparison to any published result.** Not acceptable.
- **No proof that multi-task learning helps.** You never show single-task results.

---

**Figure 2 (Parity plot):**
- No R^2 or MAE shown on the plot. Which model made this? Not stated. No color bar label. Big scatter at extreme values — not discussed.

**Figure 3 (Uncertainty plot):**
- Only ~50 data points shown. How were they picked? Looks cherry-picked.

**Figure 4 (t-SNE of embeddings):**
- No element labels anywhere. Just colored dots. This figure tells us nothing without labels.

**Figure 5 (Error heatmap):**
- Too small to read. Which elements have the biggest errors? Not named.

---

### Section V.B — Uncertainty:

> "The uncertainty-error correlation of 0.6396 indicates that prediction uncertainty is strongly correlated with prediction error."

- **0.64 is NOT strong.** In calibration work, you want > 0.8 for "strong." 0.64 is moderate at best. Do not overstate your results.
- What kind of correlation? Pearson? Spearman? Not stated.

---

> "FE MAE @ 10% Coverage = 0.0446 eV/atom"

- Even when you pick only your 10% most confident predictions, you get 0.0446 eV/atom. Published methods get better than this on the FULL test set. Not impressive.

---

### Section V.C — Volume Prediction:

- **MAE = 0.640 A^3/atom:** For typical volumes of 10-20 A^3/atom, this is a 3-6% error. For structure work, a 5% volume error gives ~1.7% error in lattice parameter. For a = 4 A, that is ~0.07 A error — too big for useful structure prediction.
- **Uncertainty-Error Correlation = 0.52:** Weak. Your uncertainty estimates are barely useful.
- **No baseline comparison.** What does a simple formula using average atomic radius give?

**Figure 6 (Volume parity plot):**
- Big outliers at high volumes (>80 A^3/atom) — model underpredicts. Not discussed.

---

### Section V.D — Space Group Prediction:

**Table VII:**
- **Top-1 accuracy = 77.2%:** Looks decent for 230 classes, but the data is very skewed (Fig. 9). A few space groups dominate. What is the accuracy on rare space groups? A model that always guesses the top 5 space groups could get >50%.
- **Venkatraman & Carvalho [10] report >80%.** Your best is worse but you never mention this.

---

> "Random Forest models outperform neural networks..."

- You say this is because RF can learn non-linear patterns. **But neural networks can do that too.** The real reason is probably that tree models work better on tabular data, or that your neural network was badly tuned. MLP at 38.6% is very suspicious.

---

> "The model systematically confuses structurally similar orthorhombic or monoclinic symmetries..."

- **This is the key finding but you bury it in one sentence.** This is not a bug — it is a basic limit of composition-only prediction. Many space groups cannot be told apart by composition alone. This needs a full paragraph of honest discussion.

---

**Figure 9:** X-axis labels unreadable. How many of the 230 space groups are in your data? Not stated.

**Figure 10:** Feature names hard to read. No discussion of what they mean physically.

**Figure 11 (Confusion matrix):** Completely unreadable. Not ready for publication.

**Figure 12 (Sensitivity/Specificity):** Also hard to read.

---

### Section V.E — Wyckoff Prediction:

**Table VIII:**
- **Micro-F1 = 0.85, Macro-F1 = 0.73:** The 12-point gap means the model works well on common Wyckoff letters but badly on rare ones.
- **Recall@K = 0.99 at K=13:** You need 13 guesses to find the right answer. For a material with 3 Wyckoff sites, 13 guesses is not useful.

---

> "The dataset contained 210,579 crystal structures"

- This is bigger than the thermodynamic dataset (150,200). How? Not explained.

---

**Figure 13:** Shows letters appearing <10 times. But you said you filtered rare patterns. What was the cutoff?

---

### Section V.F — Overall Summary:

**Table IX picks the best model for each task:**
- Formation Energy: Uncertainty Ensemble
- Hull Distance: Feature-Based MTL
- Volume: UQ Volume Model
- Space Group: RF + Crystal System
- Wyckoff: XGBoost

Every task uses a different model. **This is NOT a unified framework.** It is a collection of separate models. Be honest about this.

---

> "To rigorously evaluate the physical realism..."

- Comparing letter frequencies is NOT a rigorous test. A model that just memorizes the overall frequency pattern would pass this test. Real validation means: build a structure, relax it with DFT, check if it is correct.

---

**Figure 14 (Wyckoff Letter Fidelity):**
- Bottom panel shows ratios from 0 to 2.5. Some letters over-predicted by 150%, others missed. Most letters fall OUTSIDE the +/-20% tolerance band.
- **The text says "remarkably well." The figure shows the opposite.**

---

### Section V.G — Discussion:

This section just repeats the results in different words. No real thinking.

**Things that should be discussed but are missing:**

1. **Polymorphism:** How do you handle compositions with multiple possible structures?
2. **Error cascade:** Space group is 77.2% correct. What happens to Wyckoff prediction when the space group is wrong?
3. **Published baselines:** How do your numbers compare to existing work?
4. **Where does the model fail?** Which materials or structures are hardest?
5. **Did you check even ONE predicted structure with DFT?**

---

> "Tree-based models achieved higher prediction accuracy compared to neural networks when predicting space group's symmetry due to the ability to learn the non-linear relationships..."

- Neural networks also learn non-linear patterns. This reasoning is wrong. The real reason is likely that tree models handle tabular (Magpie) features better, or the neural network was not tuned well.

---

> "space group's symmetry conditions", "Wyckoff's position"

- Wrong grammar. These are not possessives. Write "space group symmetry" and "Wyckoff positions."

---

## 8. CONCLUSION (Section VI)

> "The predicted properties include formation energy, energy above hull, relative stability, volume per atom, space group, and Wyckoff positions."

- Still mixing thermodynamic properties with structural properties as if they are the same thing.

---

> "The proposed method will allow for large-scale screening of materials..."

- **"Will allow"** — future tense. You have not shown this. No case study. No new material found. No DFT check. This is a promise, not a result.

---

## 9. FIGURES

- **Figure 1:** The SrTiO3 example is cherry-picked because it is cubic — the only crystal system where volume gives you the lattice parameter. Try a monoclinic example and the method breaks.
- **Figures 4, 5, 9, 10, 11, 12:** All have readability problems. Labels too small, text unreadable, missing labels. Not ready for publication.
- **No figure shows the full pipeline working end-to-end.** 14 figures and none of them shows the paper's main claim actually working.

---

## 10. REFERENCES

Only **17 references** for a paper like this. Far too few.

**Missing important references:**

- **Bartel et al.** — About data leakage in ML for formation energy
- **CrabNet (Wang et al.)** — Strong composition-based model to compare against
- **Roost (Goodall & Lee)** — You cite it [4] but never compare to it
- **ALIGNN (Choudhary & DeCost)** — Important baseline
- **USPEX, CALYPSO, AIRSS** — The real crystal structure prediction methods
- **Ward et al. (2016)** — The Magpie paper. You use Magpie everywhere but never cite it
- **Anything about polymorphism** — you ignore this topic even though it is the core challenge

---

## 11. SUMMARY OF FLAWS

### FATAL FLAWS (must fix or paper cannot be readable):

1. **Title says "Crystal Structure Generation" but no structure is ever made.** Wyckoff letters without free parameters and volume without lattice parameters cannot give you a crystal structure (except for cubic).
2. **Polymorphism is never discussed.** Same composition can give different structures. This is the core challenge and you ignore it.
3. **No comparison with any published result.** You say "competitive" but compare against nothing.
4. **No end-to-end test.** You test each model alone but never run the full pipeline and measure total error.

### MAJOR FLAWS:

5. Dataset split is 80/20 in one place and 80/10/10 in another.
6. Four different dataset sizes (150K, 101K, 210K, 151K) with no explanation.
7. Claim that volume alone gives lattice parameters — wrong for non-cubic systems.
8. Claim that Wyckoff letters give atomic coordinates — wrong without free parameters.
9. No DFT validation of any prediction.
10. Multi-task loss (Eq. 6) has no weights on different tasks.
11. Stability threshold is vague ("typically close to 0 eV"). State the exact number.
12. Refs [3] and [4] are cited wrongly — they are composition-only methods but you say they need crystal structures.
13. Six model variants with no ablation study.
14. No single-task baseline to prove multi-task learning helps.

### MINOR FLAWS:

15. Typos: "mrethod", "density of functional theory."
16. Feature count changes: "130-140" then "138."
17. Correlation of 0.64 called "strong" (it is moderate).
18. "Physics-informed" used wrongly.
19. Discussion just repeats results.
20. Contribution #5 in future tense.
21. "etc." in a feature list — not OK in a paper.
22. Most figures are too small to read.
23. "space group's symmetry" and "Wyckoff's position" are wrong grammar.
24. Figure 14 shows poor agreement but text says "remarkably well."

---

## 12. WHAT TO DO NEXT

1. **Fix the title.** Either remove "Generation" or actually generate structures and check them with DFT.
2. **Compare your results to published work.** Add a table with your numbers next to Roost, CrabNet, and the space group papers you cite.
3. **Talk about polymorphism.** How do you handle compositions with multiple possible structures?
4. **Test the full pipeline end-to-end.** Chain all four models and report the total error.
5. **Fix the physics mistakes.** Volume alone does not give lattice parameters for non-cubic systems. Wyckoff letters without free parameters do not give coordinates.
6. **Add missing details.** Exact stability threshold, exact feature count, exact dataset sizes, Materials Project version, split method, all hyperparameters.
7. **Add simple baselines.** Average atomic radius for volume, majority class for stability, published results for all tasks.
8. **Rewrite in your own words.** Remove the LLM language. Add real scientific thinking and honest discussion of what does NOT work.
9. **Fix all figures.** Readable labels, element names on t-SNE, readable confusion matrix.
10. **Add more references.** At least 30-40 for a paper like this.

---

**Bottom line:** This paper is a collection of standard ML models run on Materials Project data. There is no new method, no comparison with existing work, no physical check, and the main claim (structure generation) is not true. The paper needs major rework: (1) be honest about what the paper actually does, (2) compare against published results, (3) talk about polymorphism, (4) test the full pipeline with error build-up, (5) fix the crystallography mistakes in Sections IV.F and IV.G, and (6) rewrite everything in your own words with real depth.
