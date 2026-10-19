# Split vacancy screening of the Materials Project

Data accompanying the paper:

> S. R. Kavanagh, "Identifying split vacancy defects with machine-learned foundation models and electrostatics",
> *J. Phys.: Energy* **7**, 045002 (2025). [doi:10.1088/2515-7655/ade916](https://doi.org/10.1088/2515-7655/ade916)

A split vacancy is a vacancy–interstitial–vacancy complex. Candidate cation split vacancies were screened across the Materials
Project (MP): 111,931 of the 155,361 MP entries queried in November 2024. Candidates were first screened by electrostatics (ES),
and then relaxed with the MACE machine-learned foundation model.

## Contents

| Path | Description |
|---|---|
| `MP_ES_MACE_lean_lookup_table.csv.gz` | One row per screened material (111,931): `mpid`, `formula`, `spg_num` (space group number), and whether split vacancies are predicted (see below) |
| `MP_ES_MACE_verbose_lookup_table.csv.gz` | The same materials, with the counts behind these predictions, both totals and per cation (dict-valued columns) |
| `MP_Screened_Split_Vacancies/<formula>-<mpid>.json.gz` | Lean per-material data files (111,931): the primitive structure, and for each cation the candidate split vacancy configurations with their sites, energies, symmetries and the MACE predictions |

Prediction columns of the lean lookup table (each `True` if it holds for at least one cation):

| Column | Meaning |
|---|---|
| `ES_pred_candidate_split_vacancies` | A candidate split vacancy has an electrostatic energy lower than, or within 10% of, that of the lowest-energy single vacancy |
| `MACE_pred_candidate_split_vacancies` | The lowest-energy MACE-relaxed candidate split vacancy is more than 0.05 eV lower in energy than the lowest-energy single vacancy |
| `MACE_exhaustive_pred_candidate_split_vacancies` | A MACE-relaxed candidate that remains a split vacancy lies less than 0.35 eV above (or below) the lowest-energy single vacancy |

See the paper for details of the screening workflow and these criteria.

## Usage

```python
import pandas as pd

# keep_default_na=False keeps the formula "NaN" (sodium nitride) as a string:
lean_df = pd.read_csv("MP_ES_MACE_lean_lookup_table.csv.gz", keep_default_na=False)

from monty.serialization import loadfn

data = loadfn("MP_Screened_Split_Vacancies/Ga2O3-mp-1243.json.gz")  # decodes pymatgen objects
```

In each per-material file:

- The top level holds `key` (`<formula>-<mpid>`), `formula`, `mpid`, `composition`, `elements`, `spg_num`, `spg_symbol`,
  `primitive_structure` and `ES_bulk_supercell_energy`, plus one dict per cation element.
- Each cation dict holds:
  - `split_vacancies_energy_dict`: the candidate split vacancies, keyed by electrostatic energy (eV). Each entry has the two vacancy
    sites and the interstitial site (`vac_defect_1_site`, `vac_defect_2_site`, `interstitial_site`), the vacancy–interstitial
    distances, point symmetries (`*_symm`), site multiplicities (`*_multiplicity`), and MACE relaxed energies
    (`MACE_<model>_f32_GOQN`).
  - MACE relaxations of the single vacancies (`MACE_<model>_f32_GOQN_<key>_<cation>_v_...`).
  - The summary fields `ES_*` and `MACE_*`.

  `<model>` is `small` for all materials, plus `medium` and `large` for a subset.

## Related data

- Full per-material data, including the initial and MACE-relaxed structures and DFT energies where calculated:
  [doi:10.5281/zenodo.15830541](https://doi.org/10.5281/zenodo.15830541).
- Notebooks and scripts, including an example analysis notebook (`Zenodo_Split_Vacancy_Analysis.ipynb`) and the script that
  computed the stored symmetries and multiplicities, plus the raw DFT calculations:
  [doi:10.5281/zenodo.14499358](https://doi.org/10.5281/zenodo.14499358).

## Changes

- October 2026:
  - Key names made consistent across all files: every MACE key now has a single `MACE_` prefix, and every cation uses the `ES_*`
    names. The values are unchanged.
  - The primitive structure of `PrCuGe-mp-1219977` was corrected.
  - The lookup tables were added.

## Licence and citation

This data is released under the [Creative Commons Attribution 4.0 International licence](LICENSE) (CC BY 4.0), as are the Zenodo
datasets. If you use it, please cite the paper above.
