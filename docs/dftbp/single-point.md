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
- The official [DFTB+ recipes](https://dftbplus-recipes.readthedocs.io/en/latest/index.html)
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

Now that you are familiar with the input file structure and the geometry format used,
it's time to perform your first calculation using DFTB+. For now, we will use a 
non-periodic structure, on which we will perform a static calculation. A good 
starting point is a water molecule, for which the geometry was already provided above.
Save that to a file called and **H2O.gen** and then create the input file 
**dftb_in.hsd** containing the lines below:

``` txt title="dftb_in.hsd"
Geometry = GenFormat {
    <<< "H2O.gen"
}

Driver = { }

Hamiltonian = xTB {
    Method = "GFN1-xTB"
}
```

That is all you need for a single-point calculation! The `Geometry` tag specifies the
geometry that you work with and takes as argument the reader needed to parse the file.
You could use xyzFormat or VaspFormat for the other file types. The `Driver` tag is 
reserved for the update method you used: you can perform optimizations or molecular 
dynamics runs, both of which require updated positions. Leaving it empty tells DFTB+ 
that the geometry will not be update, so you end up having a single-point calculation. 
The `Hamiltonian` tag takes as argument the type of Hamiltonian used in the 
simulations, which can either be `DFTB` or `xTB`. Finally, we specify the method used
using the `Method` tag, which is set to `GFN1-xTB`. 

All is left to do is running the simulation using the method shown above.
Congratulations! You performed your first DFTB+ simulation. The calculation resulted
in a few files being created in the local directory, which should now look like this:

```
band.out  charges.bin  detailed.out  dftb_in.hsd  dftb.out  dftb_pin.hsd  H2O.gen
```

## Inspecting the results

The final part of this tutorial is inspecting the results of the simulation, which
are split between multiple files. More information about each file can be found in 
Chapter 3 of the DFTB+ manual. Moreover, a review of the files can also be found on
[DFTB+ Recipes](https://dftbplus-recipes.readthedocs.io/en/latest/basics/firstcalc.html).

### band.out

The **band.out** file contains information about the energy levels in the system.
The first line shows the k-point number (in this case 1), the spin considered (1 for
up, 2 for down), and the weight at that specific k-point (also 1 here). After this,
the index, energy, and occupation of each orbital is provided. For a periodic
calculation, this block would be repeated for each k-point used. Similarly, for a 
spin unrestricted calculation the block would be repeated two times, once for each
spin. The information in this file is key to producing the band diagram of a 
structure, something which will be explained in a later tutorial.

``` txt title="band.out"
KPT            1  SPIN            1  KWEIGHT    1.0000000000000000
    1   -19.732  2.00000
    2   -15.422  2.00000
    3   -15.136  2.00000
    4   -13.302  2.00000
    5    -6.579  0.00000
    6    -4.194  0.00000
    7    -1.895  0.00000
    8     0.513  0.00000
```

In the case at hand, there are 8 molecular orbitals (2 from each hydrogen and 4 from 
the oxygen), 4 of which are fully occupied. Looking at the energy difference between 
the highest occupied molecular orbital (HOMO) and lowest unoccupied molecular orbital 
(LUMO), we get that the HOMO-LUMO gap of water is around 6.7 eV, which is close to 
the 6.3 eV gap reported by [Stephanie ten Brinck et al.](https://doi.org/10.1021/acsearthspacechem.1c00433)
using ZORA-BLYP-D3(BJ)/TZ2P. However, as in our case the molecule was not optimized, 
it is highly likely that the actual HOMO-LUMO gap is even higher.

### charges.bin

Unfortunately, the **charges.bin** file is in binary format, which means it is not
human-readable. DFTB+ offers the option to save in text format, but that is not
enabled by default. A complete overview of the file is provided in Appendix L of the
DFTB+ manual, but what is important to know is that this file is needed to restart 
calculations, as it stores the previously computed charges.

### detailed.out

The **detailed.out** file contains the most *detailed* information about your system:
the total charge, the partial charges, the shell occupations, energetic properties, 
the dipole moment, and many other properties that are not enabled by default. In
terms of content importance, it is on the same level as the **dftb.out** file,
meaning that you will most likely have to interact with it relatively often. The only
reason it is not higher on the list is because the information in **detailed.out**
only cover the properties of the last system analyzed, so it does not track the
changes in electronic structure between different SCC cycles or geometries. 

Since the file dumps a lot of information on you, it is important to know what you 
can find in there. The file starts off with some information about the electronic 
filling method used and the current geometry step, and then provides information
about the current state:

```
********************************************************************************
 iSCC Total electronic   Diff electronic      SCC error
    9   -0.56656703E+01    0.60625955E-07    0.28145263E-05
********************************************************************************
```

The text above mentions that on the 9th cycle of the self convergent charge (SCC)
procedure the total electronic energy was -5.67 Ha, which was only 6E-8 Ha different
compared to the 8th cycle. However, DFTB calculations do not use traditional
self convergent field (SCF) methods, and as such the energy difference is not 
important during the calculation. The important metric is the SCC error, which
tells you the difference in partial charges between the two cycles. Unconverged 
charges will result in poor results, so this tolerance has to be determined carefully.

The file then continues with information about the total charge inside the system,
and the atomic partial charges, which are based on Mulliken analysis. The Mulliken 
analysis is enabled by default for all calculations, but can be manually turned off
if it is not required.

```
Total charge:    -0.00000000      

Atomic gross charges (e)
Atom           Charge
   1      -0.59242485
   2       0.29621242
   3       0.29621242
```

The Mulliken analysis section continues with more in-depth information about the
electronic occupation, providing 3 levels of details. The first level describes
the number of electrons in the system and the electronic occupation per atom. A
quick look at the number of electrons may already raise question marks: why are there
only 8 electrons in the system when in actuality there should be 10? In the case at
hand, GFN1-xTB uses a reduced basis set where core electrons are ignored. This means
that oxygen only has the shells 2s and 2p, losing two electrons in the process. As
such, you can think that on atom 1 the population is actually 8.59, not 6.59, thus
increasing the total number of electrons to 10.

```
Nr. of electrons (up):      8.00000000
Atom populations (up)
 Atom       Population
    1       6.59242485
    2       0.70378758
    3       0.70378758
```

The second level describes the occupation per l-shell, which gives two entries per
atom (oxygen has 2s and 2p, hydrogen has 1s and 2s). The third level provides
the population per orbital, which cannot exceed 2.0 due to Pauli's exclusion
principle.

```                                                                                      
l-shell populations (up)
 Atom Sh.   l       Population
    1   1   0       1.84212929
    1   2   1       4.75029555
    2   1   0       0.66937366
    2   2   0       0.03441392
    3   1   0       0.66937366
    3   2   0       0.03441392

Orbital populations (up)
 Atom Sh.   l   m       Population  Label
    1   1   0   0       1.84212929  s
    1   2   1  -1       1.32398491  p_y
    1   2   1   0       1.42631065  p_z
    1   2   1   1       2.00000000  p_x
    2   1   0   0       0.66937366  s
    2   2   0   0       0.03441392  s2
    3   1   0   0       0.66937366  s
    3   2   0   0       0.03441392  s2
```

Followed by the Mulliken analysis, the file contains information about the energy
inside the system, such as the Fermi level, thermal smearing, total energy, and so
on. In general, the Mermin free energy is recommended as a reliable metric for the
energy inside the system, as it also accounts for electronic temperature.

```
Fermi level:                        -0.3653155827 H           -9.9407 eV
Band energy:                        -4.6740022127 H         -127.1861 eV
TS:                                  0.0000000000 H            0.0000 eV
Band free energy (E-TS):            -4.6740022127 H         -127.1861 eV
Extrapolated E(0K):                 -4.6740022127 H         -127.1861 eV
Input / Output electrons (q):      8.0000000000      8.0000000000 

Energy H0:                          -5.7249282333 H         -155.7832 eV
Energy SCC:                          0.0592579110 H            1.6125 eV
Total Electronic energy:            -5.6656703223 H         -154.1707 eV
Repulsive energy:                    0.0000000000 H            0.0000 eV
Total energy:                       -5.6656703223 H         -154.1707 eV
Extrapolated to 0:                  -5.6656703223 H         -154.1707 eV
Total Mermin free energy:           -5.6656703223 H         -154.1707 eV
Force related energy:               -5.6656703223 H         -154.1707 eV
```

The file closes off with information about the dipole moment of the system, which
in the case at hand differs from the experimentally determined 1.85 Debye. However,
this is most likely due to the settings used for the simulation and the
non-optimized geometry used, and will be improved in further tutorials.

```
Dipole moment:   -0.00000000    1.52590939   -0.00000000 au
Dipole moment:   -0.00000000    3.87847511   -0.00000000 Debye
```

Enabling more options in the `Analysis` block can extend this file to contain CM5
charges, forces, stress, and much more. Nevertheless, you are now aware of the
default content that can be found here.

### dftb.out

DFTB+ Recipes already has an in-depth explanation of the results presented in the
**dftb.out** file, which can be found [here](https://dftbplus-recipes.readthedocs.io/en/latest/basics/firstcalc.html).
In short, the header shows information about the software and how to cite it. The
following section presents the simulation settings such as the SCC tolerance, the 
maximum number of SCC iterations, the electronic solver, and so on. These should
correspond to the input from **dftb_in.hsd**. The next part shows the SCC convergence
and some energetic properties for the input geometry; this part is short for
single-point calculations but can get very lengthy for geometry optimizations and 
molecular dynamics. The file ends with some timing benchmarks that might be useful
when measuring the speed of the software.

### dftb_pin.hsd

The final file that was created (actually first chronologically) is the 
**dftb_pin.hsd** file. It is a detailed copy of out **dftb_in.hsd** file, with
the only difference being that the default settings are explicitly mentioned. This
has no value from a results point of view, but it provides a great way to check
which are all possible settings. It is especially useful since the DFTB+ manual
does not explicitly mention some settings for extended tight-binding calculations.

That is it for now! Later, we will look into what settings should be changed to
obtain accurate results.



