# Mutation-Impact Molecular Dynamics

This repository contains a Jupyter notebook for replica-based molecular dynamics simulations comparing a wild-type protein with a specified mutant.

The current notebook implements a guarded workflow for **soluble proteins** using GROMACS, PDBFixer, MDAnalysis, and a CHARMM36m/TIP3P-style setup. It reports structural ensemble differences; RMSD, RMSF, SASA, and radius of gyration are not free-energy estimates.

## Contents

```text
md-simulation/
├── MD_simulation_pipeline.ipynb  # Main workflow
├── MD_Simulation_Guide.pdf       # Extended methodology guide
├── environment.yml               # Conda/mamba environment
├── mdp_templates/                # EM, NVT, NPT, and production settings
├── uploads/                      # Input structures
└── results/                      # Generated trajectories and analyses
```

## Requirements

- Linux or another environment with a working GROMACS installation
- Python 3 with the packages in `environment.yml`
- NVIDIA GPU is optional but recommended for production runs
- A prepared PDB structure with a known biological assembly and chain numbering

Create the environment:

```bash
mamba env create -f environment.yml -n md-sim
mamba activate md-sim
```

Verify the GROMACS installation:

```bash
gmx --version
```

## Run the Notebook

Start Jupyter from the repository root:

```bash
jupyter lab
```

Open `MD_simulation_pipeline.ipynb` and execute the cells in order.

The workflow is:

1. Configure the input PDB, mutation, force field, temperature, salt concentration, and replica count.
2. Validate the target chain, residue number, and wild-type residue identity.
3. Prepare WT and mutant structures with PDBFixer.
4. Retain biological heterogens by default.
5. Build independent solvated replicas with GROMACS.
6. Run energy minimization, NVT equilibration, NPT equilibration, and production MD.
7. Correct periodic boundaries and fit trajectories for analysis.
8. Compare replica-level RMSD, RMSF, SASA, radius of gyration, and mutation-neighborhood behavior with summary statistics.
9. Optionally calculate mutation-neighborhood contact occupancy when a validated biological assembly is configured.

## Main Configuration

The configuration cell defines values such as:

```python
PDB_FILE = "uploads/ppox/3NKS_WT.pdb"
SYSTEM_TYPE = "soluble"
TARGET_CHAIN_ID = "A"
MUTATION_STR = "ALA-449-THR"
N_REPLICAS = 3
SIMULATION_TIME_NS = 100
FORCE_FIELD = "charmm36m"
WATER_MODEL = "tip3p"
```

Use at least three independent replicas for comparative analysis. The default 100 ns run is exploratory. Set `SIMULATION_TIME_NS = 1000` for a paper-comparable 1 microsecond production campaign after validating a short test run.

For oligomeric systems, provide `BIOLOGICAL_ASSEMBLY_FILE` and `OLIGOMER_CHAINS`. Contact analysis uses a configurable 5 Å residue-center cutoff and writes replica-level occupancy tables.

## Biological Components

Heterogens are not removed automatically. Review and parameterize every retained ligand, cofactor, metal, structural water, partner protein, or nucleic acid before production.

The generic notebook branch intentionally stops for:

- Membrane proteins
- Ligand-bound systems
- Metal- or cofactor-dependent systems
- Protein complexes
- Systems requiring substantial disorder or loop remodeling

For membrane systems, use a validated membrane builder such as CHARMM-GUI with appropriate lipid composition, orientation, hydration, and semi-isotropic pressure coupling. For ligands, cofactors, and metals, use validated parameters and document protonation, charge, oxidation, and coordination choices.

## Interpretation

The notebook calculates structural ensemble descriptors with replica-level variability. Do not interpret a difference in RMSD, RMSF, SASA, radius of gyration, or contact occupancy as a folding or binding free-energy change.

For mutation stability or binding effects, use a separate validated alchemical workflow such as FEP, TI, BAR/MBAR, or pmx. Folding stability requires an appropriate folded/unfolded thermodynamic cycle; binding requires both apo and complex legs. MM/GBSA is not implemented in the notebook and should only be added as an explicitly validated comparative screening step.

## Reproducibility

Record the following with each study:

- Input structure and biological assembly
- Chain and residue numbering
- Protonation and pH assumptions
- Force-field and GROMACS versions
- Ligand, cofactor, and metal parameter sources
- Replica seeds and simulation lengths
- All `grompp` warnings and their disposition
- Equilibration and convergence diagnostics

Generated trajectories and energy files can be large. Check `.gitignore` before committing simulation output.

## Additional Documentation

See [MD_Simulation_Guide.pdf](MD_Simulation_Guide.pdf) for the extended methodology and [MD_simulation_pipeline.ipynb](MD_simulation_pipeline.ipynb) for the executable workflow.