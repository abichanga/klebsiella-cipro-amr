# Ciprofloxacin resistance in *Klebsiella pneumoniae*

Which genes go with ciprofloxacin resistance in *Klebsiella pneumoniae*, and can they predict it? This is a Python notebook that runs top to bottom in Google Colab, using public genome data from BV-BRC.

Computing Bioinformatics group project, Strathmore University.


## Research question

Which resistance determinants (acquired genes and gyrA/parC hits) are associated with ciprofloxacin resistance in *K. pneumoniae* genomes from BV-BRC, and can a classifier built on them predict resistant vs susceptible isolates better than a majority-class baseline?

## Objectives

1. Describe how common each resistance determinant is in resistant and susceptible genomes.
2. Test which determinants are significantly associated with resistance (FDR corrected).
3. Train and evaluate a classifier against a baseline, and identify which features drive its predictions.

**Hypothesis:** gyrA/parC hits will be the strongest predictors, and adding plasmid-borne genes will improve accuracy further.

## Data

All data is public and comes from the [BV-BRC](https://www.bv-brc.org) REST API:

- `genome_amr`: ciprofloxacin results for *K. pneumoniae* (taxon 573).
- `sp_gene`: resistance gene annotations (source: NDARO).

| Step | Count |
|---|---|
| Records for *K. pneumoniae* and ciprofloxacin | 84,170 |
| From laboratory methods (the rest are computer predictions) | 8,701 |
| Genomes with a clear Resistant or Susceptible label | 6,127 |
| Genomes that also have gene annotation (final dataset) | 4,487 |

The final table has 4,487 genomes (2,894 resistant, 1,593 susceptible) and 54 resistance features, each marked 0 or 1 for absent or present. Features present in fewer than 45 genomes were dropped.

**Annotation fix.** 1,640 genomes had no NDARO annotation at all. For these, a missing gene meant "not checked", not "absent", so they were removed before any analysis.

## Notebook stages

| Stage | What it does |
|---|---|
| 1 | Question and objectives |
| 2 | Data acquisition from the BV-BRC API |
| 3 | Preprocessing and quality control |
| 4 | Exploratory analysis (class balance, gene prevalence, PCA) |
| 5 | Core analysis: Fisher's exact test, Benjamini-Hochberg FDR, odds ratios |
| 6 | Biological interpretation, compared with published work |
| 7 | Machine learning: baselines, logistic regression, random forest, ablation |
| 8 | Conclusions, limitations and future steps |

## Main results

- 51 of the 54 features had an odds ratio above 1. gyrA and parC hits had the strongest links to resistance (odds ratios 61 and 76), followed by the OmpC porin and the plasmid genes AAC(6')-Ib-cr and QnrB10.
- Only about half of resistant genomes carry a gyrA or parC hit.
- Test set (898 genomes, stratified 80/20 split):

| Model | Accuracy |
|---|---|
| Always guess resistant | 0.645 |
| gyrA or parC hit | 0.673 |
| 10 or more resistance genes | 0.880 |
| Logistic regression | 0.919 |
| Random forest | 0.927 |

- The models beat a guess but only beat the gene-count rule by 4 to 5 points. An ablation showed that gyrA, parC and three plasmid genes alone reach 0.864, and the other 49 features alone reach 0.901. Most of the signal is overall resistance gene count.
- Hypothesis: partly supported. gyrA and parC were the strongest single markers, and plasmid genes did add accuracy, but gyrA and parC alone find only half of resistant genomes.

## How to run

1. Click the klebsiella_notebook
2. Run all cells in order (**Runtime, Run all**).
3. The notebook downloads the data from the BV-BRC API, so no data files are needed. It also mounts Google Drive to save intermediate files. It writes to `/content/drive/MyDrive/klebsiella_project/data`, so create that folder or change `DATA_DIR` in the notebook.

Libraries used: pandas, numpy, matplotlib, scipy, statsmodels, scikit-learn and requests. All are available in Colab by default.

## Limitations

- Related genomes and clones can sit in both the training and test sets, so scores may look better than they would on new strains.
- Genes for other drug classes travel with resistance, so the data cannot separate cause from company.
- BV-BRC reports that a gyrA or parC hit exists, not which mutation it is.
- Labels come from many studies, and public genome data leans away from Africa, so results may not carry over to Kenyan strains.
- About 65% of genomes are resistant, so accuracy alone is generous. Baselines are reported for this reason.
- Some odds ratios rest on genes found in few genomes. LEN, OKP-B and OmpF were more common in susceptible genomes and are left unexplained.

## Future work

- Split by lineage so no clone appears in both training and test sets.
- Extract the exact gyrA and parC mutations from the sequences.
- Check species with average nucleotide identity.
- Test on an independent dataset, ideally including Kenyan genomes.

## Background reading

- bioRxiv preprint on ciprofloxacin resistance in *K. pneumoniae* genomes: https://www.biorxiv.org/content/10.1101/2025.09.24.678318
- Ismah et al., 2018, fluoroquinolone nonsusceptibility in *K. pneumoniae*: https://pmc.ncbi.nlm.nih.gov/articles/PMC5826109
- Study of Hungarian bloodstream isolates, *Frontiers in Microbiology*, 2019: https://www.frontiersin.org/articles/10.3389/fmicb.2019.00157/pdf
