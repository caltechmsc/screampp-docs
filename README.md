# SCREAM++ Usage Guidelines (v0.2.0-alpha.1)

## Introduction

**SCREAM++** is an enhanced, high-performance software package for automated protein side-chain placement. It builds upon the scientific foundation of the original SCREAM (Side-Chain Rotamer Excitation Analysis Method), which predicts accurate side-chain conformations using rotamer libraries and a unique flat-bottom potential strategy. This new generation of SCREAM has been completely re-engineered from the ground up in **Rust** for superior memory safety, performance, and modern development practices.

The core mission of SCREAM++ is to provide a robust, reliable, and easy-to-use tool for researchers in computational biology, structural biology, and drug design.

### Where Does SCREAM++ Fit in the Workflow?

For newcomers to computational structural biology, it's helpful to see where a tool like SCREAM++ is used. A typical workflow looks like this:

**X-ray Crystallography → Structure Preparation → Side-Chain Placement (SCREAM++) → Further Analysis (e.g., Docking)**

1.  **Get a Structure**: Scientists determine a protein's 3D structure using methods like X-ray crystallography, resulting in a PDB file.
2.  **Prepare the Structure**: This raw structure needs cleaning. This involves removing water molecules, adding missing hydrogen atoms, and fixing potential issues.
3.  **Optimize Side-Chains (SCREAM++)**: This is where SCREAM++ shines. Even after preparation, the positions of flexible side-chains might not be in their most stable, lowest-energy state. SCREAM++ takes the prepared structure and repacks the side-chains to find a more accurate and energetically favorable conformation.
4.  **Downstream Applications**: A well-optimized structure is crucial for subsequent steps, such as simulating how a drug molecule (ligand) might bind to the protein (molecular docking).

## Quick Start Guide

This guide provides a step-by-step walkthrough for performing a standard side-chain placement on a sample protein structure.

### 1. Prerequisites

Before starting, you must download the SCREAM++ command-line interface (CLI) binary suitable for your operating system.

- **Download the CLI**: Visit the [SCREAM++ GitHub Releases Page](https://github.com/caltechmsc/screampp/releases) and download the latest executable (`scream` or `scream.exe`).

### 2. Prepare Sample Files

For this tutorial, we will use a prepared BGF (Biosym Graphics File) of [PDB ID `1A8D`](https://www.rcsb.org/structure/1A8D). This is the core structure of the gp41 protein from the Human Immunodeficiency Virus (HIV-1), which is crucial for how the virus infects cells. For this experiment, the protein sample was produced using _Escherichia coli_ as an expression system.

- **Download Sample Structure**: [An example protein structure input](https://github.com/caltechmsc/screampp-docs/blob/main/input.bgf).

- **Create a Configuration File**: Create a file named `config.toml` and paste the following content into it. This configuration instructs SCREAM++ to optimize all residues using a standard forcefield and rotamer library.

```toml
# config.toml: A basic configuration for side-chain placement (v0.2 format).

# Path to the residue topology definition file. 'default' is a logical name
# that points to the standard registry downloaded by the CLI.
topology-registry-path = "default"

[forcefield]
# The 's-factor' controls the extent of the flat-bottom potential.
# A value around 1.0 is generally optimal for libraries with ~1.0 Å diversity.
s-factor = 1.1

# Logical names for the forcefield and delta parameter files.
# These will be resolved from the local data directory.
forcefield-path = "lj-12-6@0.4"
delta-params-path = "rmsd-1.0"

[sampling]
# Logical name for the rotamer library to use.
# Format is 'charge_scheme@diversity'.
rotamer-library = "charmm@rmsd-1.0"

[optimization]
# The number of final, unique, low-energy solutions to generate.
num-solutions = 1

[residues-to-optimize]
# Specifies which residues to modify. 'all' considers every residue in the system
# that has a corresponding entry in the rotamer library.
type = "all"
```

### 3. Step-by-Step Execution

#### Step 3.1: Setup and Verification

Create a new directory for your project. Place the downloaded `scream` executable, `input.bgf`, and `config.toml` inside this directory. Open your terminal or command prompt and navigate to this directory.

First, verify that the CLI is working by running the help command:

```bash
./scream --help
```

You should see a list of available commands and options, confirming the executable is functional.

#### Step 3.2: Download Required Data

The logical names used in `config.toml` (e.g., `lj-12-6@0.4`, `charmm@rmsd-1.0`) refer to data files that must be present locally. Download the standard data package using the following command:

```bash
./scream data download
```

This will fetch and unpack the forcefields, rotamer libraries, and topology files into a system-specific data directory. You only need to do this once.

#### Step 3.3: Run the Placement

The input file `input.bgf` may contain suboptimal or unrefined side-chain conformations.
<img width="992" height="771" alt="image" src="https://github.com/user-attachments/assets/1fc37742-cdea-4821-abb5-db493361b777" />
_(Imagine Figure 1 here: A visualization of the `input.bgf` structure, with red circles highlighting several side-chains in sterically unfavorable positions.)_

Now, execute the side-chain placement workflow with the following command:

```bash
./scream place -i input.bgf -o optimized.bgf -c config.toml
```

- `-i input.bgf`: Specifies the input structure file.
- `-o optimized.bgf`: Specifies the output file for the resulting structure.
- `-c config.toml`: Specifies the configuration file to use for the run.

The process will start, showing a progress bar as it calculates energies and resolves clashes.

#### Step 3.4: Analyze the Result

Upon completion, a new file, `optimized.bgf`, will be created. This file contains the same protein backbone but with the side-chains repacked into a new, lower-energy conformation.
<img width="992" height="771" alt="image" src="https://github.com/user-attachments/assets/41e21346-eb24-428d-a1cf-64588a60862d" />
_(Imagine Figure 2 here: A visualization of the `optimized.bgf` structure, showing the same highlighted regions as Figure 1, but now with the side-chains neatly packed and clashes resolved.)_

You can now use this optimized structure for further analysis or experiments.

#### Step 3.5: Visual Comparison

To better illustrate the improvement, compare the original and optimized structures side-by-side:
<img width="992" height="771" alt="image" src="https://github.com/user-attachments/assets/ee0f0c57-d52a-4792-b6f3-bc894edb627e" />
_(Imagine Figure 3 here: Overlay of the original structure in green and the optimized structure in blue. The green regions show the initial side-chain positions, while the blue regions highlight the optimized, clash-free conformations.)_

---

## Advanced Usage: Configuration Guide

The `config.toml` file provides fine-grained control over the SCREAM++ workflow. Below is a detailed explanation of the primary sections and their parameters.

### `topology-registry-path`

This top-level setting defines the residue topology for the entire run.

- `topology-registry-path` (String): Path or logical name (`"default"`) for the TOML file that defines the topology of each residue (i.e., which atoms are backbone vs. sidechain).

### `[forcefield]`

This section controls the energy function used for scoring.

- `s-factor` (Float): The scaling factor for the flat-bottom potential. It modulates the `delta` value (`Δ = μ + s ⋅ σ`). This is the most critical parameter for tuning accuracy based on the coarseness of the rotamer library.
- `forcefield-path` (String): Path or logical name (e.g., `"lj-12-6@0.4"`) for the file containing VDW and H-bond parameters.
- `delta-params-path` (String): Path or logical name (e.g., `"rmsd-1.0"`) for the CSV file containing the `mu` and `sigma` values for each atom type, used to calculate the `delta` for the flat-bottom potential.

### `[sampling]`

This section defines the conformational search space.

- `rotamer-library` (String): Path or logical name (e.g., `"charmm@rmsd-1.0"`) for the rotamer library file. The name typically indicates the charge scheme and the diversity metric (e.g., RMSD).

### `[optimization]`

This section controls the optimization algorithm.

- `num-solutions` (Integer): The number of top-scoring, unique conformations to save at the end of the run. Default is `1`.
- `max-iterations` (Integer): The maximum number of iterations for the clash resolution phase. Default is `100`.
- `include-input-conformation` (Boolean): If `true`, the original side-chain conformation from the input file is added to the pool of rotamers for sampling. Default is `false`.
- `simulated-annealing` (Table, Optional): If this section is present, a simulated annealing phase is run after clash resolution to explore a wider conformational space.
  - `initial-temperature` (Float): Starting temperature for annealing.
  - `final-temperature` (Float): Temperature at which to stop annealing.
  - `cooling-rate` (Float): Multiplicative cooling factor (e.g., `0.95`).
  - `steps-per-temperature` (Integer): Number of Monte Carlo steps at each temperature.
- `final-refinement-iterations` (Integer): Number of final local optimization passes. Default is `2`.

### `[residues-to-optimize]`

This section specifies which residues' side-chains will be modified.

- `type` (String): The method for selecting residues. Can be one of:

  1.  **`"all"`**: Optimizes all protein residues found in the rotamer library.

      ```toml
      [residues-to-optimize]
      type = "all"
      ```

  2.  **`"list"`**: Specifies residues to include or exclude explicitly.

      ```toml
      [residues-to-optimize]
      type = "list"
      # Optimize residues 12 and 15 on chain A.
      include = [
          { chain-id = 'A', residue-number = 12 },
          { chain-id = 'A', residue-number = 15 },
      ]
      # Exclude residue 50 on chain B from optimization.
      exclude = [
          { chain-id = 'B', residue-number = 50 },
      ]
      ```

  3.  **`"ligand-binding-site"`**: Optimizes all protein residues within a certain radius of a specified ligand. Note the `kebab-case` for field names.
      ```toml
      [residues-to-optimize]
      type = "ligand-binding-site"
      # Define the ligand's location.
      ligand-residue = { chain-id = 'L', residue-number = 301 }
      # Define the radius in angstroms from any heavy atom of the ligand.
      radius-angstroms = 5.0
      ```
