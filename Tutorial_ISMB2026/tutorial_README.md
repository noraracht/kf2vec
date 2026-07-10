# kf2vec

Alignment-free phylogenetic placement using k-mer frequency vectors and machine learning.

---

# Installation

## Install from Bioconda

```bash
conda create -n kf2vec_env python=3.11
conda activate kf2vec_env

conda install -c bioconda kf2vec
```

Verify installation:

```bash
kf2vec --version
kf2vec --help
```

---


# Quick Start Tutorial

The toy dataset included with kf2vec demonstrates the complete workflow:

```text
Backbone genomes
      │
      ▼
1. Extract k-mer frequencies
      │
      ▼
2. Split tree into subtrees
      │
      ▼
3. Train classifier
      │
      ▼
4. Classify queries
      │
      ▼
5. Train distance models
      │
      ▼
6. Predict query-to-backbone distances
      │
      ▼
7. Place queries onto the phylogeny
```

## Step 0. Obtain dataset

```
git clone https://github.com/noraracht/Tutorial_ISMB_data
```
---

## Step 1. Extract k-mer frequencies

```bash
kf2vec get_frequencies \
    -input_dir ./clade_11_fna_train \
    -output_dir ./clade_11_fna_train_kf

kf2vec get_frequencies \
    -input_dir ./clade_11_fna_test \
    -output_dir ./clade_11_fna_test_kf
```

Output: `.kf` files containing normalized k-mer frequencies.

---

## Step 2. Split the phylogeny and compute true distances
We split the <u>full</u> tree but compute distance on a <u>pruned</u> tree. This is how we keep track of clade membership for query sequences.

```bash
kf2vec divide_tree \
    -tree clade_11.nwk \
    -size 100
```

```bash
kf2vec get_distances \
    -tree ./train_tree/clade_11_prunned.nwk \
    -subtrees ./train_tree/train_set.subtrees
```

Output:

```text
train_tree.subtrees
subtree_0.di_mtrx
subtree_1.di_mtrx
...
```

---

## Step 3. Train the classifier

```bash
kf2vec train_classifier \
    -input_dir ./toy_example/train_tree_kf \
    -subtrees ./toy_example/train_tree_newick/train_tree.subtrees \
    -e 10 \
    -o ./toy_example/train_tree_models
```

Output:

```text
classifier_model.ckpt
```

---

## Step 4. Classify query genomes

```bash
kf2vec classify \
    -input_dir ./toy_example/test_kf \
    -model ./toy_example/train_tree_models \
    -o ./toy_example/test_results
```

Output:

```text
classes.out
```

---

## Step 5. Train distance models

```bash
kf2vec train_model_set \
    -input_dir ./toy_example/train_tree_kf \
    -true_dist ./toy_example/train_tree_newick \
    -subtrees ./toy_example/train_tree_newick/train_tree.subtrees \
    -e 10 \
    -o ./toy_example/train_tree_models
```

Output:

```text
model_subtree_0.ckpt
model_subtree_1.ckpt
...
```

### Single-clade example

```bash
kf2vec train_model_set \
    -input_dir ./toy_example/train_tree_kf \
    -true_dist ./toy_example/train_tree_newick \
    -subtrees ./toy_example/train_tree_newick/train_tree.subtrees \
    -clade 0 \
    -e 10 \
    -o ./toy_example/train_tree_models
```

---

## Step 6. Predict query-to-backbone distances

```bash
kf2vec query \
    -input_dir ./toy_example/test_kf \
    -model ./toy_example/train_tree_models \
    -classes ./toy_example/test_results \
    -o ./toy_example/test_results
```

Output:

```text
apples_input_di_mtrx_query_*.csv
```

These files are used directly by APPLES.

---

# Query Placement with APPLES

Install APPLES and GAPPA:

```bash
pip install apples

conda install bioconda::gappa
```

Run APPLES:

```bash
run_apples.py \
    -d ./toy_example/test_results/apples_input_di_mtrx_query_G000196015.csv \
    -t ./toy_example/train_tree_newick/train_tree.nwk \
    -f 0 \
    -b 5 \
    -o ./toy_example/placement/G000196015.jplace
```

Convert the placement to Newick format:

```bash
gappa examine graft \
    --jplace-path ./toy_example/placement/G000196015.jplace \
    --out-dir ./toy_example/placement/result
```

---

# Optional: Scale Branch Lengths

```bash
kf2vec scale_tree \
    -tree ./toy_example/train_tree_newick/train_tree.nwk \
    -factor 100
```

Output:

```text
train_tree_r100.0.nwk
```

---

# Command Reference

After completing the tutorial, refer to the full command documentation for:

- get_frequencies
- divide_tree
- get_distances
- scale_tree
- train_classifier
- classify
- train_model_set
- query
- get_chunks
- train_classifier_chunks
- train_model_set_chunks
