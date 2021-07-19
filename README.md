# RayleighTaylorInstablity
Rayleigh Taylor Instablity Case using OpenFOAM-v2012

How to run:
1. with Allrun script (run the case parallelly and do post-processing serially.)

```cpp
./Allrun
```
2. run the case parallelly and do post-processing parallelly too.

```cpp
blockMesh

decomposePar

mpirun -np 4 interFoam -parallel

reconstructPar
```
or
```cpp
blockMesh

decomposePar

mpirun -np 4 interFoam -noFunctionObjects -parallel

reconstructPar

mpirun -np 4 interFoam -postProcess -parallel
```

Intialise the alpha field with **codeStream** in 0/alpha.water

```cpp
internalField		#codeStream
    {
	codeInclude
	#{
	    #include "fvCFD.H"
	#};

	codeOptions
	#{
	    -I$(LIB_SRC)/finiteVolume/lnInclude \
	    -I$(LIB_SRC)/meshTools/lnInclude
	#};

	codeLibs
	#{
	    -lmeshTools \
	    -lfiniteVolume
	#};

	code
	#{
	    const IOdictionary& d = static_cast<const IOdictionary&>(dict);
	    const fvMesh& mesh = refCast<const fvMesh>(d.db());

	    scalarField alpha(mesh.nCells(), 0.);

	    forAll(alpha, i)
	    {
		const scalar x = mesh.C()[i][0];
		const scalar y = mesh.C()[i][1];

		// if (y >= 2.1 + 0.1*Foam::pow(cos(constant::
		// 	mathematical::pi*x)*cos(constant::mathematical::pi*x), 2))
		if (y >= 2.0 + 0.05*cos(constant::
			mathematical::pi*2.0*x))
		{
		    alpha[i] = 1.;
		}
	    }

		// writeEntry(os, alpha); for OpenFOAM-v8
	    alpha.writeEntry("", os);
	#};
    };
```

Post-process using **coded** functionObject which supports **serial and parallel runing**. Check the file system/controlDict for detail.
