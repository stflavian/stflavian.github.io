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
      ReadInitialCharges = No
      InitialCharges = {}
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
potentials. To do so, it expands the basis set of some elements, while keeping others
identical to GFN1-xTB. GFN2-xTB is the newest model available, featuring better
descriptions for halogens and hydrogen, while also using D4 dispersion corrections as
opposed to the D3 corrections used in GFN1-xTB and IPEA-xTB. The table below contains
information about the basis set used for each atomic species:

In progress!

| Element     | GFN1-xTB            | IPEA-xTB            | GFN2-xTB                |
| :---------- | :------------------ | :------------------ | :------------------     |
| H           | *n*s, (*n*+1)s      | *n*s, (*n*+1)s      | *n*s                    |
| He          | *n*s                | *n*s                | *n*s, (*n*+1)p          |
| Li, Be      | *n*s, *n*p          | *n*s, *n*p          | *n*s, *n*p              |
| B-F         | *n*s, *n*p          | *n*s, *n*p, (*n*+1)d| *n*s, *n*p              |
| Ne          | *n*s, *n*p, (*n*+1)d| *n*s, *n*p, (*n*+1)d| *n*s, *n*p, (*n*+1)d    |
| Group 1     | *n*s, *n*p          | *n*s, *n*p          | *n*s, *n*p              |
| Group 2     |                     |                     |                         |
| Group 13-18 |                     |                     |                         |
| B-F         |                     |                     |                         |
| B-F         |                     |                     |                         |
| tran. met.  | (*n*-1)d, *n*s, *n*p| (*n*-1)d, *n*s, *n*p| *n*d, (*n*+1)s, (*n*+1)p|
| Ln & An     | (*n*-1)d, *n*s, *n*p| (*n*-1)d, *n*s, *n*p| *n*d, (*n*+1)s, (*n*+1)p|

## Self-convergent charges (SCC)

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

The SCC section is probably the most importan

## Electron filling and smearing

```
Filling = Fermi {
  Temperature = 9.5004460357391702E-004
}
```

## System charges

```
Charge = 0.0000000000000000
ReadInitialCharges = No
InitialCharges = {}
```

## Spin-polarized calculations

```
SpinPolarisation = {}
```

## Implicit effects

As implicit effects, one can introduce and an electric field or solvents in the 
simulation, altering the behavior of the system. For zwitterionic systems the
introduction of solvation is recommended as it improves the convergence of the SCC
cycle.

```
ElectricField = {}
Solvation = {}
```

## Hamiltonian modifications



```
Dispersion = {}
Electrostatics = GammaFunctional {}
```

## Force calculation

The last part covers how the forces are being calculated. `ForceEvaluation` can be
performed in three ways: `traditional`, `dynamics`, and `dynamicsT0`. The first
one is the default version used to calculate forces in DFTB, while the other two
are meant to be used for non-converged charges. In each case, the differentiation
can be performed using either finite difference, or using `Richardson` extrapolation,
which is the most accurate method to determine forces.

```
ForceEvaluation = "Traditional"
Differentiation = FiniteDiff {
  Delta = 1.2207031250000000E-004
}
```
