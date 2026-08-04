# ContactsAnalysis

Notebooks for parsing Chimera contact / H-bond exports and generating residue- and secondary-structure–level contact maps across tubulin nucleotide states.

Structures are 13-protofilament hexamers. BeF₃ / AlF₃ moieties are rewritten to phosphates before contact calculation to allow correct HBond identification in ChimeraX. The GTP-cycle order used for pairwise comparisons is:

**GMPCPP → GDP·BeF₃ → GDP·AlF₃ → GDP**

## Installation

```bash
mamba create -n contacts python ipykernel pandas matplotlib openpyxl
mamba activate contacts
python -m ipykernel install --user --name contacts --display-name "contacts"
```

or alternatively: 

```bash
mamba env create -f environment.yml
mamba activate contacts
```


Select the **contacts** kernel when opening the notebooks via IDE.

## Pipeline

Run the notebooks in order:

1. **`generate_dataframe.ipynb`** — parse Chimera text exports into tidy CSVs.
2. **`generate_maps_and_heatmaps.ipynb`** — build unified interaction edges, residue-pair difference maps, and SS-element heatmaps.

### Inputs

| File | Role |
|------|------|
| `contacts/contacts-all13pf-swapAtoms.txt` | Chimera contacts export |
| `contacts/hbonds-all13pf-loose-swapAtoms.txt` | Chimera H-bonds (0.1 Å / 10° extra slack) |
| `alpha-tubulinSS.txt`, `beta-tubulinSS.txt` | Secondary-structure element ranges according to J Löwe et al, 2001|

ChimeraX contact lists can be generated using the `contacts` and `hbonds` commands as follows: 

```python
contacts interModel f saveFile <filename>
hbonds interModel false angleSlop 30 distSlop 0.5 saveFile <filename>
```


### Intermediate / outputs

| Path | Description |
|------|-------------|
| `contacts-all13pf.csv`, `hbonds-all13pf.csv` | Parsed atom-level tables |
| `unified_edges_df-13pf-swapAtoms.csv` | Combined contacts + H-bonds edge table |
| `pairwise_diff_summary_unified.csv` | Net gained/lost interactions per transition × interface |
| `contact_maps_differences/` | Residue–residue difference maps |
| `heatmaps/` | SS contact / change SVGs (hbonds, non-polar, water-mediated, combined) |
| `ss_contact_count_analysis3.xlsx` | SS-level counts and deltas |

## Interfaces

Hexamer chain pairs analyzed:

| Label | Chains | Type |
|-------|--------|------|
| A→B | A–B | intradimer |
| A→C | A–C | interdimer |
| A→D | A–D | α lateral |
| C→F | C–F | β lateral |

## Contact counting rules

Chimera contacts and H-bonds are the source of truth. Downstream aggregation:

- **H-bonds** — every bridging H-bond is counted.
- **Hydrophobic (non-polar)** — strictly C–C contacts within VdW distance; counted as binary (at most one count per residue pair).
- **Water-mediated** — taken only from the H-bond file (not from the contacts export). A water-mediated link is a residue pair connected through at least one shared water.

Residue–residue difference maps and SS heatmaps report absolute counts per state and deltas along the nucleotide transitions above.

Notebooks tested using software versions listed in environment.yml on Mac OS X 15.6.1