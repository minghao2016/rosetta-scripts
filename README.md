# ROSETTA Scripts
Some Rosetta Scripts that allow for various tasks.

## 1. RELAX (relax.py)
**Performs a relaxation protocol for energy minimization of a PDB structure.** 

Spawns a defined number of decoys (100 by default)
that perform the protocol in parallel, using positional restraints on the backbone atoms (Turned on by default).
The restraints weight can also be set (1.0 by default).
Simulation results are saved in a .fasc file that can be analyzed to find the best structures (See `get_best.py` script).
Use `relax.py -h` for more detailed information regarding the optional arguments of this script.

## 2. DESIGN (design.py)
**Performs a design protocol by randomly mutating the aminoacid sequence in a target region in order to minimize the energy
of the structure.**

The design effort will be localized in the surrounding aminoacids (with a cutoff of 9.0 angstrom) of a ligand peptide,
defined as a single chain (B, by default) of the input PDB structure. Spawns a defined number of decoys (100 by default) that
perform the protocol in parallel. A cycle of design is comprised of two steps: (1) Monte Carlo design using a rotamer packer;
(2) Minimization step; The whole protocol is comprised of several of these cycles (4 by default).
Simulation results are saved in a .fasc file that can be analyzed to find the best structures (See `get_best.py` script).
Use `relax.py -h` for more detailed information regarding the optional arguments of this script.

## 3. MULTI DOCK (multi_dock.py)
**Performs a Position And Sequence Simultaneous Optimization (PASSO) protocol from multiple starting positions.**

The PASSO protocol intends to simultaneously optimize both the position of part of a protein
(for example, a new grafted domain) and design the interface region between the movable and fixed parts.
This protocol expects an **"ABC Model"**, where the target protein is divided in 3 different chains on the input PDB file:

1. Chain A - Fixed part of the protein
2. Chain B - Fixed part of the protein + target ligand. Will be used to define the designable region of the protein.
3. Chain C - Movable part of the protein

See the `single_dock_decoy.py` script docstring for more information on how to set up ABC models using the
`ze_util/molecule_manipulation.py` tools.

**PASSO protocol:** A single step of the PASSO protocol is comprised of 3 steps:

1. Random placement of the movable block using a rigid body movement
(translation of maximum 0.5 ansgtrom + rotation of maximum 3 degrees);
2. Evaluation of a pre filter (See the PreFilter class docstring on `ze_utils/pyrosetta_classes.py`
for more detailed information);
3. *If the newly proposed structure passes the pre filter on step 2* design the interface residues
(Uses the design protocol from the `design.py` script);

**Reporter files:** A total of 3 files will be created when running this protocol, in addition to structural PDBs:

1. Run data file (.JSON): holds information regarding each of the steps performed by the protocol.
2. Energy file (.DAT): holds energy information regarding the accepted steps whose design stage produced mutants
with energy values lower than the initial conformation.
3. Status file (.TXT): a single line file that gets overwrited every step.
Simply holds information regarding the current step and the maximum number of steps of this simulation.

See the PASSO class docstring on `ze_utils/pyrosetta_classes.py` for more detailed information.

**Docking grid:** The `multi_dock.py` script attempts to initialize this conformational search process from multiple
different starting positions for the movable part of the protein, each defined as a point in space on a grid. By default,
the grid is centered on a virtual point on the vector between chain A and chain B centroids, at the same distance from
chain B centroid as the chain C centroid. See the DockingGrid class docstring on `ze_utils/pyrosetta_classes.py` for more
detailed information.

**SLURM parallelization:** This protocol is performed by calling the `single_dock.py` script, who is in turn responsible
for spawning multiple (100 by default) decoys of `single_dock_decoys.py` scripts. If the SLURM flag is set when launching
the `multi_dock.py` script, individual slurm bash scripts are automatically created and added to the slurm queue. This
release of new decoys is controlled by a check step that verifies that there is enough space on the user queue
(limited to a set number of queued jobs - 500 by default).

## 4. ANALYZE (analyze.py)
**Analizes a single PDB file and outputs revelant intra-pose metrics.**

1. Total energy
2. Individual residues energy per selection
3. Interaction energy metric (If a ligand is present)
4. Solvent Acessible Surface Area (SASA) Metric
5. Hydrogen bonding network energy

**NOTE: This script is still under development.**

## 5. ALIGN and MULTI_ALIGN (align.py and multi_align.py)
**`align.py` script allows the alignment and RMSD calculation of two PDB files, while `multi_align.py` applies the same
protocol to all the 1 on 1 combinations for all the PDB files in the current working directory.**

Optinally, a subset of atoms can be dined by atom name (such as CA only, for example). 
Makes use of `ze_util/molecule_manipulation.py` tools.

## 6. GET BEST STRUCTURES FROM FASC FILES (get_best.py)
**Reads a .fasc file and prints the best structures.**

By default, uses 'total_score' parameter for structure comparison, and prints the n structures (5 by default) with the
**lowest** values for the parameter. If the reverse flag is set to True, print the n structures with the **highest**
values instead.

## 7. STATUS (status.py)
**Used to verify the current status of a `multi_dock.py` script running.**

By default prints the completion percentage of each docking grid point and the n structures (5, by default) with the *lowest*
'total_score' parameter. If used in a SLURM environment, print the current percentage of rescources being used by the user.