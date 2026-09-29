# MD Simulation Platform

GPU-accelerated molecular dynamics simulation environment for mutant protein models.

## System Requirements (Detected)
- **GPU:** NVIDIA GeForce RTX 3090 (24 GB VRAM, CUDA 13.2)
- **CPU:** Intel Core i7-11700 @ 2.50GHz (16 cores)
- **RAM:** 62 GB

---

## Quick Start

### 1. Create the environment & launch UI
```bash
cd /home/kmu/Taimoor/Genomics/research_bioinformatics/md-simulation
./launch.sh
```
Then open **http://localhost:8501** in your browser.

### 2. Manual environment setup (if needed)
```bash
mamba env create -f environment.yml -n md-sim
conda activate md-sim
streamlit run app/main.py
```

---

## Project Structure

```
md-simulation/
├── environment.yml          # Mamba environment (GROMACS, OpenMM, MDAnalysis, Streamlit…)
├── launch.sh                # One-command launcher
├── app/
│   ├── main.py              # Streamlit entry point + navigation
│   └── pages/
│       ├── page_home.py         # Dashboard & system info
│       ├── page_upload.py       # PDB upload & management
│       ├── page_configure.py    # Force field, box, MD parameters
│       ├── page_run.py          # Launch simulation + live logs
│       ├── page_analysis.py     # RMSD, RMSF, Rg, H-bonds, DSSP, energy
│       ├── page_visualization.py# Interactive 3D viewer (py3Dmol)
│       └── page_results.py      # Browse, download, compare, delete runs
├── scripts/
│   ├── gmx_runner.py        # GROMACS pipeline (pdb2gmx→EM→NVT→NPT→MD)
│   └── analysis.py          # MDAnalysis + MDTraj analysis functions
├── mdp_templates/
│   ├── em.mdp               # Energy minimization
│   ├── nvt.mdp              # NVT equilibration
│   ├── npt.mdp              # NPT equilibration
│   └── md.mdp               # Production MD (10 ns default)
├── uploads/                 # Uploaded PDB files
├── results/                 # Simulation output directories
└── logs/                    # Log files
```

---

## Simulation Pipeline

```
Upload PDB → pdb2gmx → editconf → solvate → genion
           → Energy Minimization (EM)
           → NVT Equilibration (100 ps)
           → NPT Equilibration (100 ps)
           → Production MD (10 ns default, configurable)
           → Analysis (RMSD, RMSF, Rg, H-bonds, DSSP, Energy)
```

All steps run GPU-accelerated via GROMACS with:
- `-nb gpu` — non-bonded on GPU
- `-pme gpu` — PME electrostatics on GPU
- `-bonded gpu` — bonded forces on GPU
- `-update gpu` — coordinate update on GPU

---

## Key Tools Installed

| Tool | Purpose |
|------|---------|
| GROMACS 2024 | Main MD engine (GPU) |
| OpenMM ≥8.1 | Python-native GPU MD |
| AmberTools | tleap, antechamber (ligand params) |
| MDAnalysis | Trajectory analysis |
| MDTraj | Fast trajectory analysis + DSSP |
| NGLView | Interactive 3D viewer |
| Streamlit | Web UI |
| Plotly | Interactive graphs |

---

## Force Fields Available

- `charmm36m` — Best for proteins, membranes (default)
- `amber99sb-ildn` — Widely used for proteins
- `amber14sb` — Latest AMBER protein FF
- `gromos54a7` — United-atom
- `oplsaa` — All-atom, good for organic molecules

---

## Running Multiple Mutants

Each simulation run is stored in `results/<job_name>_<timestamp>/`. Use the **Results Browser** page to compare RMSD curves across mutants side-by-side.

---

## Manual GROMACS Commands

```bash
conda activate md-sim

# Check GROMACS installation
gmx --version

# Run pipeline manually
python scripts/gmx_runner.py

# Extract energy data
gmx energy -f results/<run>/md.edr -o energy.xvg

# Convert trajectory to PDB
gmx trjconv -f results/<run>/md.xtc -s results/<run>/md.tpr \
            -o results/<run>/traj.pdb -pbc mol -ur compact
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| GROMACS not found | `conda activate md-sim` |
| GPU out of memory | Reduce box size or use `-gpu_id ""` for CPU |
| pdb2gmx fails | Check PDB for missing atoms, use `-ignh` to rebuild H |
| Pressure coupling error | Ensure NVT ran to completion before NPT |
