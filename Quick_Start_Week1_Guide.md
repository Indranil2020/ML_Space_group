# QUICK START GUIDE
## Week 1 Action Items for ML Materials Database Project

---

## 🎯 YOUR FIRST WEEK GOALS

By the end of Week 1, you should have:
1. ✅ All software installed
2. ✅ Downloaded test data from at least one database
3. ✅ Run your first ML model (baseline)
4. ✅ Team roles assigned

---

## 📥 DAY 1: SETUP (Monday)

### Software Installation

#### Step 1: Create conda environment

```bash
# Create new environment
conda create -n materials-ml python=3.10
conda activate materials-ml

# Essential packages
pip install numpy pandas matplotlib seaborn scikit-learn

# Materials science packages
pip install pymatgen mp-api jarvis-tools matminer

# Machine learning (choose one to start)
pip install torch  # OR pip install tensorflow

# Utilities
pip install jupyter notebook tqdm
```

#### Step 2: Get API keys

**Materials Project:**
1. Go to https://next-gen.materialsproject.org/
2. Create free account
3. Go to "API" section
4. Copy your API key
5. Save it: Create file `~/.config/mp_api_key.txt`

**Test it works:**
```python
from mp_api.client import MPRester

with MPRester("YOUR_API_KEY") as mpr:
    fe2o3 = mpr.materials.summary.search(formula="Fe2O3")
    print(f"Found {len(fe2o3)} Fe2O3 entries")
```

---

## 📚 DAY 2: DATA EXPLORATION (Tuesday)

### Download Your First Dataset

Create `01_download_data.py`:

```python
"""
Download 1000 test materials from Materials Project
"""

from mp_api.client import MPRester
import pandas as pd
import pickle

# Your API key
API_KEY = "YOUR_KEY_HERE"

print("Downloading data from Materials Project...")

with MPRester(API_KEY) as mpr:
    # Search criteria: binary and ternary oxides
    docs = mpr.materials.summary.search(
        num_elements=(2, 3),
        elements=["O"],  # Must contain oxygen
        num_sites=(1, 50),  # Not too large
        fields=[
            "material_id",
            "formula_pretty", 
            "composition",
            "symmetry",
            "formation_energy_per_atom",
            "energy_above_hull",
            "band_gap",
            "volume",
            "density",
            "is_stable"
        ],
        max_results=1000
    )
    
print(f"Downloaded {len(docs)} materials")

# Convert to DataFrame
data = []
for doc in docs:
    data.append({
        'material_id': doc.material_id,
        'formula': doc.formula_pretty,
        'composition': str(doc.composition),
        'space_group': doc.symmetry.number,
        'crystal_system': doc.symmetry.crystal_system,
        'formation_energy': doc.formation_energy_per_atom,
        'e_hull': doc.energy_above_hull,
        'band_gap': doc.band_gap,
        'volume': doc.volume,
        'density': doc.density,
        'is_stable': doc.is_stable
    })

df = pd.DataFrame(data)

# Save
df.to_csv('data_mp_1000.csv', index=False)
df.to_pickle('data_mp_1000.pkl')

print("\nDataset saved!")
print(df.head())
print(f"\nSpace group distribution:")
print(df['space_group'].value_counts().head(10))
```

**Run it:**
```bash
python 01_download_data.py
```

---

## 🔬 DAY 3: FEATURE ENGINEERING (Wednesday)

### Create Features from Compositions

Create `02_featurize.py`:

```python
"""
Convert compositions to ML features
"""

import pandas as pd
from matminer.featurizers.composition import ElementProperty
from pymatgen.core import Composition
from tqdm import tqdm

# Load data
print("Loading data...")
df = pd.read_pickle('data_mp_1000.pkl')

# Convert to Composition objects
print("Converting to Composition objects...")
df['comp_obj'] = df['composition'].apply(Composition)

# Featurize with Magpie
print("Featurizing (this takes ~2 minutes)...")
ep = ElementProperty.from_preset("magpie")

features_list = []
for comp in tqdm(df['comp_obj']):
    try:
        features = ep.featurize(comp)
        features_list.append(features)
    except:
        # Some compositions might fail
        features_list.append([None] * len(ep.feature_labels()))

# Create feature DataFrame
df_features = pd.DataFrame(features_list, columns=ep.feature_labels())

# Combine with original data
df_final = pd.concat([df.reset_index(drop=True), df_features], axis=1)

# Remove rows with missing features
df_final = df_final.dropna(subset=ep.feature_labels())

print(f"\nFinal dataset: {len(df_final)} materials with {len(ep.feature_labels())} features")

# Save
df_final.to_pickle('data_featurized.pkl')
print("Saved to data_featurized.pkl")

# Quick analysis
print("\nFeature statistics:")
print(df_features.describe())
```

**Run it:**
```bash
python 02_featurize.py
```

---

## 🤖 DAY 4: FIRST ML MODEL (Thursday)

### Train Space Group Predictor

Create `03_train_baseline.py`:

```python
"""
Baseline space group prediction model
"""

import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report
import matplotlib.pyplot as plt

# Load featurized data
print("Loading data...")
df = pd.read_pickle('data_featurized.pkl')

# Prepare features and target
feature_cols = [col for col in df.columns if col.startswith('MagpieData')]
X = df[feature_cols].values
y = df['space_group'].values

print(f"Features: {X.shape[1]}")
print(f"Samples: {len(y)}")
print(f"Unique space groups: {len(np.unique(y))}")

# Train/test split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

print(f"\nTrain: {len(X_train)} samples")
print(f"Test: {len(X_test)} samples")

# Train Random Forest
print("\nTraining Random Forest...")
model = RandomForestClassifier(
    n_estimators=100,
    max_depth=20,
    min_samples_split=5,
    random_state=42,
    n_jobs=-1
)
model.fit(X_train, y_train)

# Evaluate
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)

print(f"\n{'='*50}")
print(f"Test Accuracy: {accuracy:.3f}")
print(f"{'='*50}")

# Top-5 accuracy
y_pred_proba = model.predict_proba(X_test)
top5_correct = 0
for i, true_sg in enumerate(y_test):
    top5_indices = np.argsort(y_pred_proba[i])[-5:]
    top5_classes = model.classes_[top5_indices]
    if true_sg in top5_classes:
        top5_correct += 1

top5_acc = top5_correct / len(y_test)
print(f"Top-5 Accuracy: {top5_acc:.3f}")

# Feature importance
feature_importance = pd.DataFrame({
    'feature': feature_cols,
    'importance': model.feature_importances_
}).sort_values('importance', ascending=False)

print("\nTop 10 Important Features:")
print(feature_importance.head(10))

# Plot
plt.figure(figsize=(10, 6))
plt.barh(feature_importance.head(15)['feature'], 
         feature_importance.head(15)['importance'])
plt.xlabel('Importance')
plt.title('Top 15 Features for Space Group Prediction')
plt.tight_layout()
plt.savefig('feature_importance.png', dpi=150)
print("\nSaved feature_importance.png")

# Save model
import pickle
with open('model_baseline.pkl', 'wb') as f:
    pickle.dump(model, f)
print("Saved model_baseline.pkl")
```

**Run it:**
```bash
python 03_train_baseline.py
```

**Expected output:**
```
Test Accuracy: 0.350-0.450
Top-5 Accuracy: 0.650-0.750
```

---

## 📊 DAY 5: ANALYSIS & PLANNING (Friday)

### Analyze Your Results

Create `04_analysis.ipynb` (Jupyter notebook):

```python
# Cell 1: Load and explore
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_pickle('data_featurized.pkl')

# Space group distribution
plt.figure(figsize=(12, 6))
sg_counts = df['space_group'].value_counts().head(20)
plt.bar(range(len(sg_counts)), sg_counts.values)
plt.xlabel('Space Group')
plt.ylabel('Count')
plt.title('Top 20 Most Common Space Groups')
plt.xticks(range(len(sg_counts)), sg_counts.index, rotation=45)
plt.tight_layout()
plt.show()

# Cell 2: Stability analysis
stable = df[df['e_hull'] < 0.025]
print(f"Stable materials: {len(stable)} ({len(stable)/len(df)*100:.1f}%)")

plt.figure(figsize=(10, 6))
plt.hist(df['e_hull'], bins=50, range=(0, 0.5), alpha=0.7)
plt.axvline(0.025, color='red', linestyle='--', label='Stability threshold')
plt.xlabel('Energy Above Hull (eV/atom)')
plt.ylabel('Count')
plt.title('Distribution of Hull Distances')
plt.legend()
plt.show()

# Cell 3: Formation energy vs space group
plt.figure(figsize=(12, 6))
top_sgs = df['space_group'].value_counts().head(10).index
df_top = df[df['space_group'].isin(top_sgs)]
df_top.boxplot(column='formation_energy', by='space_group', figsize=(12, 6))
plt.ylabel('Formation Energy (eV/atom)')
plt.title('Formation Energy by Space Group')
plt.suptitle('')
plt.tight_layout()
plt.show()

# Cell 4: Feature correlations
feature_cols = [col for col in df.columns if col.startswith('MagpieData')]
corr_with_sg = df[feature_cols + ['space_group']].corr()['space_group'].sort_values(ascending=False)
print("Features most correlated with space group:")
print(corr_with_sg.head(10))
```

**Run Jupyter:**
```bash
jupyter notebook 04_analysis.ipynb
```

---

## 👥 TEAM MEETING (Friday afternoon)

### Agenda

**30 minutes: Show & Tell**
- Everyone presents what they did this week
- Show your plots and results
- Discuss challenges

**30 minutes: Planning**
- Review full proposal document
- Assign roles for next 2 weeks:
  - **Person 1:** Expand database (add JARVIS, OQMD)
  - **Person 2:** Implement CrabNet model
  - **Person 3:** Set up DFT workflow
  - **Person 4:** Database design

**15 minutes: Next week goals**
- Set specific deliverables for Week 2
- Schedule next meeting

---

## ✅ WEEK 1 CHECKLIST

### Setup
- [ ] Python environment created
- [ ] All packages installed
- [ ] Materials Project API key obtained
- [ ] Tested API connection

### Data
- [ ] Downloaded 1000 test materials
- [ ] Explored data structure
- [ ] Understand space groups, formation energy, hull distance

### ML
- [ ] Created features using Matminer
- [ ] Trained baseline Random Forest
- [ ] Achieved >30% accuracy (top-1), >60% (top-5)
- [ ] Understand feature importance

### Analysis
- [ ] Created visualizations
- [ ] Analyzed space group distribution
- [ ] Analyzed stability patterns
- [ ] Identified interesting materials

### Team
- [ ] All team members completed Week 1 tasks
- [ ] Had team meeting
- [ ] Roles assigned for Week 2
- [ ] Communication channel set up (Slack/Discord)

---

## 🚨 COMMON ISSUES & SOLUTIONS

### Issue 1: API download too slow

**Solution:** Use smaller batches
```python
# Instead of max_results=1000
max_results=100  # Start small

# Or use pagination
for i in range(0, 1000, 100):
    docs = mpr.materials.summary.search(..., skip=i, limit=100)
```

### Issue 2: Featurization crashes

**Solution:** Add error handling
```python
for comp in df['comp_obj']:
    try:
        features = ep.featurize(comp)
    except Exception as e:
        print(f"Failed for {comp}: {e}")
        features = [np.nan] * len(ep.feature_labels())
```

### Issue 3: Low accuracy

**Solution:** This is expected! Baseline is ~35-45%. We'll improve with:
- Better models (CrabNet, deep learning)
- More training data
- Better features
- Ensemble methods

### Issue 4: Out of memory

**Solution:** Use smaller batches
```python
# Process in chunks
chunk_size = 100
for i in range(0, len(df), chunk_size):
    chunk = df.iloc[i:i+chunk_size]
    # Process chunk
```

---

## 📚 READING FOR WEEK 1

**Papers (skim, focus on methods):**
1. Ward et al., "A general-purpose ML framework" (2016) - Magpie features
2. Jain et al., "Materials Project" (2013) - Database overview
3. Goodall & Lee, "Predicting without crystal structure" (2020) - Roost model

**Documentation:**
- Matminer tutorials: https://github.com/hackingmaterials/matminer_examples
- Materials Project docs: https://docs.materialsproject.org/

---

## 🎯 WEEK 2 PREVIEW

### Goals
- Expand to 10,000+ materials
- Try CrabNet/Roost model
- Add JARVIS data
- Set up DFT test calculations
- Improve accuracy to >50% (top-1), >75% (top-5)

### Preparations
- Read CrabNet paper
- Install additional packages
- Get HPC access (for DFT person)
- Design database schema

---

## 💡 SUCCESS METRICS

### Minimum Viable Results (Week 1)
- ✅ 1000+ materials downloaded
- ✅ Features generated
- ✅ Baseline model: >30% accuracy
- ✅ Team organized

### Good Results (Week 1)
- ✅ 2000+ materials
- ✅ Multiple feature sets tested
- ✅ Baseline: >40% accuracy
- ✅ Initial analysis completed

### Excellent Results (Week 1)
- ✅ 5000+ materials
- ✅ Baseline + one advanced model
- ✅ Baseline: >45% accuracy
- ✅ Publication-quality figures
- ✅ Clear next steps identified

---

## 🆘 GETTING HELP

### When stuck:
1. **Check error messages carefully** - often they tell you exactly what's wrong
2. **Google the error** - materials ML is popular, likely someone had same issue
3. **Materials Project forum:** https://matsci.org/
4. **Ask your team** - someone might know!
5. **Check documentation** - PyMatGen, Matminer docs are excellent

### Useful communities:
- Materials Project forum
- PyMatGen discussions: https://github.com/materialsproject/pymatgen/discussions
- Matminer issues: https://github.com/hackingmaterials/matminer/issues

---

## 🎉 CELEBRATE!

### After Week 1, you will have:
- Working ML pipeline for materials
- Thousands of materials in your database
- Your first trained model
- Beautiful visualizations
- Clear path forward

**This is a real research project!** You're doing cutting-edge materials science. Be proud! 🏆

---

## 📞 NEXT STEPS

After completing Week 1:
1. ✅ Check all items in checklist
2. 📧 Email your results to team lead
3. 📅 Confirm Week 2 meeting time
4. 📖 Start reading for Week 2
5. 💬 Share your progress in team channel!

---

## 🚀 READY? LET'S GO!

**Day 1 (Monday):** Setup + Installation  
**Day 2 (Tuesday):** Download data  
**Day 3 (Wednesday):** Featurize  
**Day 4 (Thursday):** Train model  
**Day 5 (Friday):** Analyze + Team meeting

---

*Good luck! You've got this! 💪*

---

## APPENDIX: Complete Code Repository Structure

```
ml-materials-project/
│
├── README.md                          # Project overview
├── requirements.txt                   # Python packages
│
├── data/
│   ├── raw/
│   │   └── data_mp_1000.csv          # Downloaded data
│   └── processed/
│       └── data_featurized.pkl       # With features
│
├── scripts/
│   ├── 01_download_data.py
│   ├── 02_featurize.py
│   ├── 03_train_baseline.py
│   └── 04_analysis.ipynb
│
├── models/
│   └── model_baseline.pkl             # Trained model
│
└── figures/
    └── feature_importance.png         # Visualization
```

**Create this structure:**
```bash
mkdir -p ml-materials-project/{data/{raw,processed},scripts,models,figures}
cd ml-materials-project
```

---

## FINAL TIPS

1. **Start simple** - Don't try to do everything in Week 1
2. **Document everything** - You'll forget what you did
3. **Save your work** - Git commit regularly
4. **Ask questions** - No stupid questions in research!
5. **Have fun** - This is exciting work! 🎉

**Now go download that data and train that model!** 🚀
