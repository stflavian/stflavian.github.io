---
draft: false 
date: 2025-10-14 
categories:
  - DFTB+
---

# Performing single-point calculations in DFTB+

This tutorial explores the settings required to perform static calculations in DFTB+,
mainly using extended tight-binding (xTB) methods such as GFN1-xTB and GFN2-xTB.
The section starts with an overview of the files necessary to start a simulation, and
then goes into all the possible options one can pick as of version 24.1 of DFTB+.

The tutorial is based on three sources of information:

- The DFTB+ manual (highly recommended to have at hand), which you can find 
[here](https://github.com/dftbplus/dftbplus/releases/download/24.1/manual.pdf)
- The official [DFTB+ 
recipes](https://dftbplus-recipes.readthedocs.io/en/latest/index.html)
- My three-month experience with the software

<!-- more -->

## Folder structure

Unlike other molecular modeling software, DFTB+ requires only one file to set up a 
basic calculation: the input file, always called **dftb_in.hsd**. The complete 
specifications of the Human-friendly Structured Data (.hsd) format are presented in 
Appendix B in the DFTB+ manual, but they can be summarized as:

``` txt title="Human-friendly Structured Data format"
Property = value  # Everything after a hashtag is a comment

Property = value { subvalue1 subvalue2 }

SettingBlock = {
    Property1 [units] = value
    Property2 = value
    SubSettingBlock = Value {
        ...
    }
}

SettingBlock = {
    <<< "file.txt"
}
```

``` txt title="file.txt"
Property = value
```

The .hsd format is flexible, as it does not enforce the usage of tabs and new lines. 
One could in principle write an entire settings block on one line and DFTB+ would 
still be able to read it. Moreover, uppercase and lowercase letters are treated the 
same, so the exact case used does not affect the behavior of the software. Finally, 
it is possible to introduce raw text from external files using `<<<`, which makes the
.hsd format modular.

To start the simulation, one only needs to call `dftb+` in the local folder and the
software automatically parses the **dftb_in.hsd** file and proceeds with the 
calculations. As DFTB+ prints the results on the screen, a good idea is to redirect
the output to a file which can be analyzed later, which can be done using the `tee`
program. The recommended complete command is:

```
dftb+ | tee dftb.out
```

## Loading geometries

As you might expect, a geometry file is needed in order to perform a simulation using 
DFTB+. This geometry can either be provided as part of the **dftb_in.hsd** file, or
can be loaded from an external file using the `<<<` operator mentioned above. 
Throughout these tutorials, only the second method will be used as it is more
flexible and shortens the DFTB+ input file. Nevertheless, if you insist on having all
information in one file, you only have to replace `<<< "name.file"` with the contents 
of the geometry file.

Depending on the type of calculation that has to be performed (M - molecular, 
P - periodic), DFTB+ currently supports 4 different formats for geometries:

- Explicit geometry specification (M, P)
- .gen files (M, P)
- .xyz files (M)
- POSCAR and CONTCAR files (P)

Information on how to interact with each of these formats can be found in Section 2.2 
of the DFTB+ manual. In this tutorial we will only be focusing on the .gen file 
format, which is the intended file format to be used with DFTB+. 

The .gen file is a simple text-based format used to define the atomic structure of a 
system for simulations. The complete specification of the format are explained in
Appendix D of the DFTB+ manual. In short, it contains information about the number of 
atoms, atomic species, and their coordinates. Inspired from the .xyz format, the 
first line of the file contains the number of atoms, together with a flag. Currently, 
there are 4 flags available:

- C (from cluster) is used for non-periodic calculations
- S (from supercell) is used for periodic calculations with Cartesian coordinates
- F (from fractional) is used for periodic calculations with fractional coordinates
- H (from helical) is used for one-dimensional periodic helical cells

The second line contains a list of atomic species (element symbols), separated by 
spaces. Following that, the data for each atom is provided, which includes the atom 
index (starting from 1), atomic species index (referring to the order in the second 
line), and X, Y, and Z coordinates (in Ångströms if Cartesian). Finally, if the 
structure is periodic, the last 4 lines should provide the origin and each lattice
vector. An example file for a simple water molecule is provided below:

``` txt title="H2O.gen"
3 C
O H
1 1  0.00000000000E+00 -0.10000000000E+01  0.00000000000E+00
2 2  0.00000000000E+00  0.00000000000E+00  0.78306400000E+00
3 2  0.00000000000E+00  0.00000000000E+00 -0.78306400000E+00
```

Another example is a periodic structure written in Cartesian coordinates, in this
case a gallium arsenide crystal:

``` txt title="GaAs.gen"
2 S
Ga As
1 1  0.000000 0.000000 0.000000
2 2  1.356773 1.356773 1.356773
0.000000 0.000000 0.000000
2.713546 2.713546 0.
0.       2.713546 2.713546
2.713546 0.       2.713546
```

In case you prefer it in fractional coordinates, here is another example:

``` txt title="GaAs.gen"
2 F
Ga As
1 1  0.0  0.0  0.0
2 2  0.25 0.25 0.25
0.000000 0.000000 0.000000
2.713546 2.713546 0.
0.       2.713546 2.713546
2.713546 0.       2.713546
```

## Setting up an extended tight-binding calculation

Now 

