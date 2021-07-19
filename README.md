# RayleighTaylorInstablity
Rayleigh Taylor Instablity Case using OpenFOAM-v2012

How to run:
```cpp
./Allrun
```

Intialise the alpha field with codeStream in 0/alpha.water

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

Postprocess with coded functionObject in system/controlDict
```cpp
functions{
    VariationInY
    {
        functionObjectLibs ( "libutilityFunctionObjects.so" );
        enabled         true;
        type            coded;
        name    VariationInY;
        executeControl      runTime;
        executeInterval     0.1;
        writeControl        runTime;//timeStep;
        writeInterval       0.1;
        log true;

        codeInclude
        #{
            #include "fvCFD.H"
            #include <fstream>
            // #include "OFstream.H"
        #};
    
        codeOptions
        #{
            -I$(LIB_SRC)/finiteVolume/lnInclude \
            -I$(LIB_SRC)/meshTools/lnInclude \
            -I$(LIB_SRC)/OpenFOAM/lnInclude
        #};
    
        codeLibs
        #{
            -lmeshTools \
            -lfiniteVolume \
            -lOpenFOAM
        #};

        codeExecute
        #{
            const volScalarField& alpha
            (
                mesh().lookupObject<volScalarField>("alpha.water")
            );

            const fvMesh& mesh = alpha.mesh();

            scalar height(0.0);

            scalar xDirection(0.0);

            volVectorField position = mesh.C();

            forAll (mesh.C(), celli)
            {
                if (alpha[celli] > scalar(0.1))
                {
                    if (height < 2.0 - position[celli].y())
                    {
                        height = 2.0 - position[celli].y();
                        xDirection = position[celli].x();
                    }
                }
            }

            word currentTime = obr_.time().timeName();

            word Dir = mesh.time().path()/type();

            if(!isDir(Dir))
            {
                mkDir(Dir);
            }
            
            std::ofstream out;

            out.open(Dir/type(), std::ios::app);

            out << currentTime << "\t" << height << "\t" << xDirection<< "\n";
       #};
    }
}
```
