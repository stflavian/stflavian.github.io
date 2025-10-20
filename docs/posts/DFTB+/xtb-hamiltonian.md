---
draft: true 
date: 2025-10-17 
categories:
  - DFTB+
---

# Using the extended tight-binding Hamiltonian

Now that you know how to perform an extended tight-binding calculation (xTB) on a
non-periodic system, it is time to go more in-depth into the settings used for that
calculation. After all, the results produced by the calculation should be accurate
and reliable. Also, using the correct settings for the xTB Hamiltonian will play a
crucial part during geometry optimizations, as this can improve convergence speed
and results accuracy.

<!-- more -->

The best place to start is by having a look at the Hamiltonian block from the
**dftb_pin.hsd** file generated during our previous simulation. That way, all 
possible settings are visible. Fore more details on each entry you can check Chapter
2.4 from the DFTB+ manual.

To make the settings block more clear, the keys can be shuffled to create more
consistent blocks, which will be explained below.

=== "Unordered"

    ```
    Hamiltonian = xTB {
      Method = "GFN1-xTB"
      ShellResolvedSCC = Yes
      SCC = Yes
      ReadInitialCharges = No
      InitialCharges = {}
      SCCTolerance = 1.0000000000000001E-005
      ConvergentSCCOnly = Yes
      SpinPolarisation = {}
      ElectricField = {}
      Solver = RelativelyRobust {}
      Charge = 0.0000000000000000
      MaxSCCIterations = 100
      Dispersion = {}
      Solvation = {}
      Electrostatics = GammaFunctional {}
      Differentiation = FiniteDiff {
        Delta = 1.2207031250000000E-004
      }
      ForceEvaluation = "Traditional"
      Mixer = Broyden {
        MixingParameter = 0.20000000000000001
        InverseJacobiWeight = 1.0000000000000000E-002
        MinimalWeight = 1.0000000000000000
        MaximalWeight = 100000.00000000000
        WeightFactor = 1.0000000000000000E-002
      }
      Filling = Fermi {
        Temperature = 9.5004460357391702E-004
      }
    }
    ```

=== "Ordered"
    
    ```
    Hamiltonian = xTB {
      Method = "GFN1-xTB"
      SCC = Yes
      SCCTolerance = 1.0000000000000001E-005
      MaxSCCIterations = 100
      ShellResolvedSCC = Yes
      ConvergentSCCOnly = Yes
      Solver = RelativelyRobust {}
      Mixer = Broyden {
        MixingParameter = 0.20000000000000001
        InverseJacobiWeight = 1.0000000000000000E-002
        MinimalWeight = 1.0000000000000000
        MaximalWeight = 100000.00000000000
        WeightFactor = 1.0000000000000000E-002
      }
      Filling = Fermi {
        Temperature = 9.5004460357391702E-004
      }
      Charge = 0.0000000000000000
      InitialCharges = {}
      ReadInitialCharges = No
      SpinPolarisation = {}
      ElectricField = {}
      Solvation = {}
      Dispersion = {}
      Electrostatics = GammaFunctional {}
      ForceEvaluation = "Traditional"
      Differentiation = FiniteDiff {
        Delta = 1.2207031250000000E-004
      }
    }
    ```

## Extended tight-binding models

```
Method = "GFN1-xTB"
```

The first tag, `Method`, is used to specify the extended tight-binding method used 
for the calculations. Currently, DFTB+ supports three models:

- [GFN1-xTB](https://pubs.acs.org/doi/10.1021/acs.jctc.7b00118)
- [IPEA-xTB](https://pubs.rsc.org/en/content/articlelanding/2017/sc/c7sc00601b)
- [GFN2-xTB](https://pubs.acs.org/doi/10.1021/acs.jctc.8b01176)

Much like DFT functionals, which one is best depends on the system studied and on the
chemical species involved. GFN1-xTB is the first extended tight-binding model 
proposed by Grimme et al. and covers parameters for all atomic species with Z ≤ 86.
IPEA-xTB is a reparameterization of GFN1-xTB aimed at predicting better ionization
potentials. To do so, it expands the basis set of some elements, while keeping most
identical to GFN1-xTB. GFN2-xTB is the newest model available, featuring better
descriptions for halogens and hydrogen, while also using D4 dispersion corrections as
opposed to the D3 corrections used in GFN1-xTB and IPEA-xTB. The advantages of using 
GFN2-xTB over GFN1-xTB depend on the system, but it is generally observed that
GFN2-xTB has more convergence problems for bad geometries compared to its predecessor.
The table below contains information about the basis sets used for each atomic 
species:


|  n  |    Element   | GFN1-xTB       | IPEA-xTB       | GFN2-xTB       |
|:---:|:------------:|----------------|----------------|----------------|
|  1  |       H      | 1s, 2s         | 1s, 2s         | 1s             |
|     |      He      | 1s             | 1s             | 1s, 2p         |
|  2  |    Li, Be    | 2s, 2p         | 2s, 2p         | 2s, 2p         |
|     |      B-F     | 2s, 2p         | 2s, 2p, 3s     | 2s, 2p         |
|     |      Ne      | 2s, 2p, 3d     | 2s, 2p, 3d     | 2s, 2p, 3d     |
| 3-5 |    Group 1   | ns, nd         | ns, nd         | ns, nd         |
|     |    Group 2   | ns, np, (n-1)d | ns, np, (n-1)d | ns, np, (n-1)d |
|     |      Na*     | 3s, 3p         | 2s, 2p         | 3s, 3p         |
|     |      Mg*     | 3s, 3p         | 2s, 2p         | 3s, 3p, 3d     |
|     |  Groups 3-11 | (n-1)d, ns, np | (n-1)d, ns, np | (n-1)d, ns, np |
|     |   Group 12   | ns, np         | ns, np         | ns, np         |
|     | Groups 13-17 | ns, np, nd     | ns, np, nd     | ns, np, nd     |
|     |  Noble gases | ns, np, nd     | ns, np, nd     | ns, np, nd     |
| 6   |      Cs*     | 5s, 5p         | 5s, 5p         | 6s, 6p         |
|     |      Ba*     | 5s, 5p, 4d     | 5s, 5p, 4d     | 6s, 6p, 5d     |
|     |  Lanthanides | 5d, 6s, 6p     | 5d, 6s, 6p     | 5d, 6s, 6p     |
|     |  Groups 3-11 | 5d, 6s, 6p     | 5d, 6s, 6p     | 5d, 6s, 6p     |
|     |      Hg      | 6s, 6p         | 6s, 6p         | 6s, 6p         |
|     |     Tl-Bi    | 6s, 6p         | 6s, 6p         | 6s, 6p         |
|     |      Po      | 6s, 6p, 5d     | 6s, 6p, 5d     | 6s, 6p         |
|     |    At, Rn    | 6s, 6p, 5d     | 6s, 6p, 5d     | 6s, 6p, 5d     |

The elements that have an asterisk use incorrect basis sets in some of the three 
models. These are known bugs that were part of the original parameterizations and 
thus require significant changes to the models to correct. As such, they will remain
unchanged.

## Self-convergent charges (SCC)

The most important block in the `Hemiltonian` block is the SCC part, which directly
impacts the performance and accuracy of the calculations. These settings impact

```
SCC = Yes
SCCTolerance = 1.0000000000000001E-005
MaxSCCIterations = 100
ShellResolvedSCC = Yes
ConvergentSCCOnly = Yes
Solver = RelativelyRobust {}
Mixer = Broyden {
  MixingParameter = 0.20000000000000001
  InverseJacobiWeight = 1.0000000000000000E-002
  MinimalWeight = 1.0000000000000000
  MaximalWeight = 100000.00000000000
  WeightFactor = 1.0000000000000000E-002
}
```

The SCC section is probably the most important

## Electron filling and smearing

The `Filing` block specifies the partial occupation method and electron smearing 
used in the calculations. Currently, two methods are available: `Fermi` and 
`MethfesselPaxton`. Both methods take as argument `Temperature`, which controls the
smearing in the system much like `SIGMA` does in VASP, with the difference that 
DFTB+ uses Hartree as default units. By default, DFTB+ uses a smearing of 300 K, which
can be considered low enough for some applications. Nevertheless, according to
[Basiuk et al.](https://doi.org/10.1016/j.mtcomm.2020.101595), if HOMO-LUMO gaps
or other electronic properties are of interest, the smearing can be lowered to 30 K 
for more accurate results.

=== "DFTB+ default"

    ```
    Filling = Fermi {
      Temperature = 9.5004460357391702E-004
    }
    ```

=== "VASP default"

    ```
    Filling = MethfesselPaxton {
      Order = 1
      Temperature = 7.3498586E-003 
    }
    ```

Which of the two fractional occupation methods produces the best results depends on
the system studied. DFTB+ defaults to `Fermi` filling with 300 K smearing, while 
VASP defaults to 1st order `MethfesselPaxton` filling with 0.2 eV (2321 K) smearing.
Nevertheless, the latter can be inaccurate for insulators due to unphysical partial
occupancies. 

## System charges

This part of the input controls the charges inside the system. The `Charge` property
sets the system charge, which is important when ionic systems are (e.g. acetate, 
chloride, etc.). For neutral systems, it should be left unchanged. For ionic system,
this should be paired with the following tag, `InitialCharges`, which assigns the
correct charges to the respective atoms.

```
Charge = 0.0000000000000000
InitialCharges = {}
ReadInitialCharges = No
```

The following tag, `ReadInitialCharges`, allows for previously computed results to
be read, which provides the basis of the restart system in DFTB+. As mentioned 
previously, simulations create a **charges.bin** file that stores information about
the charges computed during the simulation. Loading these charges at start-time
would result in the simulation continuing from those charges, which is effectively
a restart.


## Spin-polarized calculations

While this will be explore in depth in another tutorial, the `SpinPolarisation` tag
enables spin-polarised calculations. By default no value is assigned to it, forcing
DFTB+ to ignore spin polarisation. By changing it to `Colinear` or `NonColinear`, 
the user can introduce polarisation in the system, which might be necessary to 
accurately model some species (e.g. manganese). 

```
SpinPolarisation = {}
```

## Implicit effects

As implicit effects, one can introduce an electric field or a solvent in the 
simulation, altering the behavior of the system. For zwitterionic systems the
introduction of solvation is recommended as it improves the convergence of the SCC
cycle. I am not currently interested in simulations with either properties, so 
unfortunately I cannot provide more information than what is already stated in the 
DFTB+ manual.

```
ElectricField = {}
Solvation = {}
```

## Hamiltonian modifications

This part of the settings provides the ability to alter the Hamiltonian used in
extended tight-binding, but in practice has little utility. While not specifically
stated anywhere, the default dispersion schemes used in GFN1-xTB, IPEA-xTB, and 
GFN2-xTB are already part of the base parameterization, and thus does not require
manual additions using the `Dispersion` tag. Meanwhile, the `Electrostatics` tag
should only be changed from `GammaFunctional` to `Poisson` during transport
calculations.

```
Dispersion = {}
Electrostatics = GammaFunctional {}
```

## Force calculation

The last part covers how the forces are being calculated during the simulation. 
`ForceEvaluation` can be performed in three ways: `traditional`, `dynamics`, and 
`dynamicsT0`. The first one is the default version used to calculate forces in DFTB, 
while the other two are meant to be used for non-converged charges. In each case, 
the differentiation can be performed using either finite difference (`FiniteDiff`), 
or using `Richardson` extrapolation, which is the most accurate method to determine 
forces.

```
ForceEvaluation = "Traditional"
Differentiation = FiniteDiff {
  Delta = 1.2207031250000000E-004
}
```
