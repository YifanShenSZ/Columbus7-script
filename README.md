# Columbus7-Extension
Useful extension and modification for Columbus7

Featured utilities:
* Various modifications to improve Columbus7 in compatibility and functionality (Modification)
* Run a batch of single point calculation (SinglePointBatch)
* Collect the batch (Collect)
* Calculate Hessian and vibration (Hessian)

See each subdirectory for details

## Complement to Columbus7 documentation
Columbus7 originates in 1980s and preserves many historical codes. Besides, the user community of Columbus7 is always small. As a result, there are compatibility issues and undiscovered bugs, which are out of the coverage of the official documentation

### Installation
Steps
1. Install a Global Arrays
2. Enter a desired directory, tar -xzvf the installation package
3. export COLUMBUS=(current directory)/Columbus
4. cp Path_To_Global_Arrays/lib/*.a $Columbus
5. vi install.config. If the last line(s) = cpan || standard || grad || parallel, then delete these line(s). Repeat this operation each time before ./install.automatic
6. ./install.automatic cpan standard grad
7. ./install.automatic parallel

Solutions to some issues
* Global Arrays (subsequently Columbus7) does not support mpi 3.0 standard, e.g. openmpi 4 fails but 3 works
* Installation may fail with intel mpi. For now openmpi always works
* Replace certain files with their counterparts in Modification

### Basis set
To use special basis, do not run prepinp

The basis file ($COLUMBUS/source/iargos/basis/*.bas) format is:
1. Basis name
2. Number of basis types
3. Number of alphas, l + 1, number of contracted basis
4. Alpha value, contraction coefficients

The basis input file (daltaoin) format is:
1. (s for spherical basis, c for Cartesian basis), type of atoms, unknown flag, unknown flag
2. Atom number, number of this type of atoms, number of l types, number of s orbitals, number of p orbitals, ...
3. Atom coordinate
4. H marking the head of a basis, number of alphas, number of contracted basis. Basis are sorted by l
5. Alpha value, contraction coefficients

infofl contains the number of basis in each irreducible representation

### MCSCF
MCSCF is a nonlinear optimization problem, so may easily be trapped into local minimum, e.g.:
* For symmetric molecule, molecular orbital may lose symmetry if not explicitly using symmetry

### MRCI
#### Computation cost
The upper limit of CI expansion on 24 core avx2 processor computer in 2019 is ~ 100,000,000

#### General modification
Tighten MCSCF tolerance in mcscfin before running MRCI, recommend:
* gradient (W) norm < tol(2)=1e-8
* orbital correction (K) norm < tol(3)=1e-8

To run parallel MRCI:
* you may have to manually change the 'ciudg' keyword in control.run to 'pciudg'
* modify nseg in ciudgin according to WORK/ciudg.perf to speed up

Tighten MRCI tolerance in ciudgin before computing gradient, recommend:
* RTOLCI = 1e-5 or 1e-6

Tighten MRCI gradient tolerance in cigrdin:
* orbital resolution < wndtol = 1e-9, wnatol = 1e-9, wnvtol = 1e-9
* final effective densitymatrix < ftol = 1e-11
* if using LAPACK solver (no symmetry): solvelpck = 1, mdir = 0, cdir = 0; else Columbus solver (with symmetry): nmiter = 200, nvrsmx = 200, rtol = 1e-10, dtol = 1e-10

#### Special skill
* FROOT: NROOT computes the lowest NROOT MRCI roots; FROOT follows the FROOT-th MCSCF root and optimizes it with MRCI (keeping tracking by overlap), then output it along with lower MRCI states. FROOT is better at giving the state you want, but may fail when MRCI has different state ordering from MCSCF

#### Common bug
'maxbl' too small: increase maxbl in cisrtin, usually up to 200000

### Geometry optimization

#### Minimum
* NROOT also specifies which surface to optimize on. You can overwrite it with the transition moment input transmomin (i.e. set computing transition moment between m-th and m-th state to optimize on m-th surface)
* By default GDIIS searches for minimum with BFGS. You should see a line in gdiisin as bfgs

#### Saddle point
* No one has ever tried RGF
* To search for saddle point with GDIIS, replace the bfgs line in gdiisin with sadd

### Conical intersection search
* "GDIIS never converges" -- Yarkony
* polyhesin to start search (to build BFGS hessian): &NACINT{maxit=200,newton=1,iheseq1=1,ihess=0,ipflg=3,accel=1,scale=0.1,kscale=0}/end
* polyhesin to end search: &NACINT{maxit=200,newton=1,iheseq1=-1,methodn=99*-1,ihess=0,ipflg=3,accel=1,scale=1.0,kscale=2,}/end. Copy old h-pieces and continuity

### Weird stuff
Bug
* ifort 2018.3 with avx2 leads to bug in mcscf.x

Interactive input (colinp)
* In CI input, display bug occurs for font size > 12
* In hessian input, perl bug occurs for font size > 8

Redundant input files generated by colinp
* infofl.*: when they are the same to infofl
* daltcomm.new: when it is the same to daltcomm

Orbital
* The number of each kind of orbitals appearing in cidrtin cannot exceed 256, including the frozen orbitals

Gradient
* If specifying 'cigrad' in control.run, then runc assumes geometry optimization job, so gdiisin is required
* If specifying 'cigrad' in control.run, then cigrdin cannot contain 'nadcalc'
* If specifying 'nadcoupl' in control.run, then sometimes calculating only energy gradients without nonadiabatic couplings would make cigrd.x raise inconsistent-energy error
