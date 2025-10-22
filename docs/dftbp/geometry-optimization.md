# Geometry optimizations

With single-point calculations covered and the xTB Hamiltonian explained, there is
only one more step needed to relax structures: moving the atoms around. Luckily, much
like any other quantum calculation method, DFTB+ comes with a variety of engines to 
optimize structures. In this section, I will explore the options available during 
the relaxation. 

As a note, DFTB+ contains some drivers meant for structure relaxations which are 
deprecated, and as such I will not mention them. To be more precise, this tutorial
only covers the `GeometryOptimization` driver.

## Optimizing a molecule

As established before, the best way to learn what can be done in DFTB+ is by
performing a basic calculation and then checking the resulting **dftb_pin.hsd**. 
Geometry optimizations are no different, and to perform one you can recycle the 
single-point input file from before and set the `Driver` to `GeometryOptimization`,
which results in:

``` txt title="dftb_in.hsd"
Geometry = GenFormat {
    <<< "H2O.gen"
}

Driver = GeometryOptimization { }

Hamiltonian = xTB {
    Method = "GFN1-xTB"
}
```

That is all! Start the simulation as before and let's check the results.

## Analyzing the output

The first thing you may notice is that the geometry optimization produced the same
files as the single-point calculation from before, except for the newly created
**geo_end.xyz** and **geo_end.gen** files. Having a quick look at them, they contain
the same geometries but using different formats. In addition to that, the .xyz file
has a forth column representing the charges of the atoms.

=== "geo_end.gen"

    ```
    3  C
    O H
    1  1   -0.2186009291E-16   -0.7125311670E+00    0.5521821461E-16
    2  2   -0.1108690679E-15   -0.1436844645E+00    0.7710042456E+00
    3  2    0.1327291608E-15   -0.1436844645E+00   -0.7710042456E+00
    ```

=== "geo_end.xyz"

    ```
    3
    Geometry Step: 9
    O     -0.00000000     -0.71253117      0.00000000      6.67118505
    H     -0.00000000     -0.14368446      0.77100425      0.66440747
    H      0.00000000     -0.14368446     -0.77100425      0.66440747
    ```

These two geometries turn out to be the relaxed structure, and all other output files
correspond to these exact geometries. As mentioned before, except for **dftb.out**, 
output files do not keep track of properties as the simulation progresses, but only
display the properties of the last computed cycle. On the other hand, **dftb.out**
now shows information about each optimization step, such as an overview of the 
SCC procedure and energetic information for the converged charges. However, you will
notice that the geometries used for each step are not saved by default, so there is 
no way to know what happened during the relaxation.

Having a look at the results, we can see from **band.out** that the HOMO-LUMO gap
of the water molecule went from the previously computed 6.7 eV to 9.3 eV, which is
considerably different from the literature value. Meanwhile, according to 
**detailed.out**, the dipole moment went down from 3.88 Debye to 2.77 Debye, which 
is closer to experimental data. 

## Possible optimization settings

The **dftb_pin.hsd** file can give us a better clue of what is actually going on 
during the optimization. I will go through the settings one by one, but as before
more details about each entry can be found in the DFTB+ manual, more specifically in
Section 2.3.1.

```
Driver = GeometryOptimization {
  Optimiser = Rational {
    DiagLimit = 1.0000000000000000E-002
  }
  LatticeOpt = No
  MovedAtoms = "1:-1"
  Convergence = {
    Energy = 1.7976931348623157E+308
    GradNorm = 1.7976931348623157E+308
    GradElem = 1.0000000000000000E-004
    DispNorm = 1.7976931348623157E+308
    DispElem = 1.7976931348623157E+308
  }
  MaxSteps = 60
  OutputPrefix = "geo_end"
  AppendGeometries = No
}
```

The `GeometryOptimization` driver starts off with setting the `Optimiser`, which is 
set to `Rational` by default. In total, there are four optimizers available:

- `Rational`, which is meant to provide stability and accuracy, but scales 
quadratically with the number of variables optimized;
- `FIRE`, which is meant to provide speed as it scales linearly with the number of
variables optimized;
- `LBFGS`, which is also meant to be fast (also linear);
- `SteepestDescent`.

While there is no answer to the question "Which optimizer should I use?", there are
some good practices that you can always apply:

1. Always try the linear optimizers (`FIRE` and `LBFGS`) before settling to 
`Rational`, as they can provide a significant speed-up to your simulations;
2. For large structures that are difficult to optimize, you can try combining 
optimizers: use something like `FIRE` with looser convergence criteria, then use 
`Rational` with tighter convergence criteria to find a stable minima;
3. Avoid using `SteepestDescent` unless you need it for a specific reason.

The setting block then continues with `LatticeOpt`, which is disabled by default. This
setting allows DFTB+ to change the cell lattice vectors to obtain a fully relaxed 
cell, but has no usage for non-periodic calculations. I will go more in depth on this
in a later tutorial.

The tag `MovedAtoms` controls which atoms are moved during the simulation, and by 
default the entire system is selected. This is useful when constrains are needed for
large molecules, or when parts of the molecule should be kept intact while others 
need adjustments. 

The `Convergence` block is the most important part of the section, as it sets the
convergence criteria for the optimization. By default, you can notice that `Energy`,
`GradNorm`, `DispNorm`, and `DispElem` all have huge values assigned; this is 
because "stock" DFTB+ optimizations only track the maximum absolute gradient element,
which has to be smaller than 0.0001 Hartree/Bohr. 

This is a good moment to introduce units in the equation: DFTB+ uses atomic units by
default, but allows the user to use custom units for many properties. Looking at 
forces, for example, there are three options available:

- **Hartree per Bohr**: Hartree/Bohr, Ha/Bohr, au
- **Electronvolts per Ångström**: eV/Angstrom, eV/AA, eV/A
- **Joules per meter**: Joule/meter, J/m

It is always a good idea to use the units you are most familiar with instead of
manually converting values. To specify the unit of the property, you would attach
it in square brackets next to the property, something like:

=== "Ha/Bohr"
    
    ```
    Convergence = {
        GradElem [Ha/Bohr] = 1.0000000000000000E-004
    }
    ```
    
=== "eV/AA"
    
    ```
    Convergence = {
        GradElem [eV/AA] = 1.0000000000000000E-004
    }
    ```

=== "J/m"

    ```
    Convergence = {
        GradElem [J/Bohr] = 1.0000000000000000E-004
    }
    ```

The same can be done for length, mass, volume, energy, force, time, charge, velocity, 
pressure, frequency, electric field strength, dipole moment, and angular units. I 
will not list all units here, but if you are curious you should check Appendix C
of the DFTB+ manual.

Coming back to the convergence criteria, choosing the appropriate thresholds is 
crucial to keeping both performance and accuracy high; using loose values can lead
to inaccurate results, while tight values lead to longer convergence times. It is
always a good idea to get some inspiration from other quantum calculation software:

