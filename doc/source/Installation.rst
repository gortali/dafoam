.. _Installation:

Installation 
------------

This section assumes you want to compile the DAFoam optimization package from the source on a Linux system. If you use the pre-compiled version, skip this section.

**DAFoam** runs on Linux systems and is based on **OpenFOAM-v1812**. You must install OpenFOAM and verify that it is working correctly. You also need to install the 3rd party and **MDOLab** packages before using DAFoam for optimization. Other dependencies include: 

- C/C++ compilers (gcc/g++ or icc/icpc)
  
- Fortran compiler (gfortran or ifort)
  
- MPI software (openmpi, mvapich2, or impi)
  
- Swig
  
- cmake

To compile the documentation, you also need:

- Doxygen 

- Graphviz

- Sphinx 

**NOTE:** Always try to use the system provided C/Fortran compiler and MPI software to compile DAFoam and all its dependencies. 
Using self-built C/Fortran and MPI may cause linking issues

|

To install the **DAFoam** package:

1. Create a "**OpenFOAM**" folder in your home directory ($HOME). Go into the "OpenFOAM" directory and install **OpenFOAM-v1812** following this website: http://openfoamwiki.net/index.php/Installation/Linux/OpenFOAM-v1806/Ubuntu **NOTE**: DAFoam does not support long integer, so in OpenFOAM/OpenFOAM-v1812/etc/bashrc, use 32bit integer (WM_LABEL_SIZE=32), and double precision scalar (WM_PRECISION_OPTION=DP). After the OpenFOAM installation is done, start a new session to install the rest packages; **DO NOT** load the OpenFOAM environment. This is to prevent environmental variable conflict between OpenFOAM and other packages.


2. Create a "**packages**" folder in your home directory. Go into the "packages" directory and install the following 3rd party packages:

- **Anaconda** Python. **NOTE**: Anaconda2-2.4.0 is recommended since the newer version may have libgfortran conflict. Download https://repo.continuum.io/archive/Anaconda2-2.4.0-Linux-x86_64.sh and run::
  
   chmod 755 Anaconda2-2.4.0-Linux-x86_64.sh && ./Anaconda2-2.4.0-Linux-x86_64.sh 

  When asked, put $HOME/packages/anaconda2 as the prefix for the installation path. At the end of the installation, reply "yes" to add the anaconda bin path to your bashrc.

- **PETSc v3.6.4** (http://www.mcs.anl.gov/petsc/). Download petsc-3.6.4.tar.gz and put it in the $HOME/packages folder, go into $HOME/packages/petsc-3.6.4, and run::

   sed -i 's/ierr = MPI_Finalize();CHKERRQ(ierr);/\/\/ierr = MPI_Finalize();CHKERRQ(ierr);/g' src/sys/objects/pinit.c

  This will comment out line 1367 in src/sys/objects/pinit.c to prevent Petsc from conflicting with OpenFOAM MPI. After this, run::

   ./configure --with-shared-libraries --download-superlu_dist --download-parmetis --download-metis --with-fortran-interfaces --with-debugging=no --with-scalar-type=real --PETSC_ARCH=real-opt --download-fblaslapack
   
  and after this, run::

    make PETSC_DIR=$HOME/packages/petsc-3.6.4 PETSC_ARCH=real-opt all

  Add the following into your bashrc and source it::

    export PETSC_DIR=$HOME/packages/petsc-3.6.4
    export PETSC_ARCH=real-opt
    export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
    export PATH=$PETSC_DIR/$PETSC_ARCH/include:$PATH
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PETSC_DIR/$PETSC_ARCH/lib
    export PETSC_LIB=$PETSC_DIR/$PETSC_ARCH/lib

- **cgnslib_3.2.1** (http://cgns.sourceforge.net/download.html). Download cgnslib_3.2.1.tar.gz, put it in the $HOME/packages/ folder, and untar it. NOTE: The 3.2.1 version fortran include file is bad, so you need to manually edit the cgnslib_f.h file in the src directory and remove all the comment lines at the beginning of the file starting with "C". Then run::

    cmake .

  and::

    ccmake .

  A “GUI” appears and toggle ENABLE_FORTRAN by pressing [enter] (should be OFF when entering the screen for the first time, hence set it to ON). Type ‘c’ to reconfigure and ‘g’ to generate and exit. Then run::

    make

  Now add this into your bashrc and source it::

    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/packages/cgnslib_3.2.1/src

- **mpi4py-1.3.1** (http://bitbucket.org/mpi4py/mpi4py/downloads/). Untar and run::
 
    python setup.py install --user
    
  This will install the package to the .local directory.
  
- **petsc4py-3.6.0** (http://bitbucket.org/petsc/petsc4py/downloads/). Untar and run::
 
    python setup.py install --user
    
  This will install the package to the .local directory.
  


3. Create a "**repos**" folder in your home directory. Go into the "repos" directory and download and install the following **MDOLab** packages (use small cases for all the repository names):

- First add this to your bashrc and source it::
 
     export PYTHONPATH=$PYTHONPATH:$HOME/repos/
   
- Get **baseClasses** (https://github.com/mdolab/baseclasses). No need to compile. 

- Get **pyGeo** (https://github.com/mdolab/pygeo). No need to compile.
 
- Get **openFoamMeshReader** (https://github.com/mdolab/openfoammeshreader). No need to compile.   

- Get **multipoint** (https://github.com/mdolab/multipoint). No need to compile.   

- Get **pySpline** (https://github.com/mdolab/pyspline). Run::
   
     cp config/defaults/config.LINUX_GFORTRAN.mk config/config.mk
   
  and::
 
     make
    
- Get **pyHyp** (https://github.com/mdolab/pyHyp). Run::
   
     cp -r config/defaults/config.LINUX_GFORTRAN_OPENMPI.mk config/config.mk
    
  In config.mk, change the flags for cgnslib to::

     CGNS_INCLUDE_FLAGS=-I$(HOME)/packages/cgnslib_3.2.1/src
     CGNS_LINKER_FLAGS=-L$(HOME)/packages/cgnslib_3.2.1/src -lcgns
   
  and::
 
     make

- Get **cgnsUtilities** (https://github.com/mdolab/cgnsutilities). Run::
   
     cp config.mk.info config.mk

  In config.mk, change the flags for cgnslib to::

     CGNS_INCLUDE_FLAGS=-I$(HOME)/packages/cgnslib_3.2.1/src
     CGNS_LINKER_FLAGS=-L$(HOME)/packages/cgnslib_3.2.1/src -lcgns

  and::
 
     make
     
  Add this to your bashrc and source it::
   
     export PATH=$PATH:$HOME/repos/cgnsutilities/bin
     
- Get **IDWarp** (https://github.com/mdolab/idwarp). Run::
     
     cp -r config/defaults/config.LINUX_GFORTRAN_OPENMPI.mk config/config.mk

  In config.mk, change the flags for cgnslib to::

     CGNS_INCLUDE_FLAGS=-I$(HOME)/packages/cgnslib_3.2.1/src
     CGNS_LINKER_FLAGS=-L$(HOME)/packages/cgnslib_3.2.1/src -lcgns

  and::
   
     make
     
- Get **pyOptSparse** (https://github.com/mdolab/pyoptsparse). Run::
 
     python setup.py install --user


4. Download **DAFoam** (https://github.com/mdolab/dafoam) and put it into the $HOME/repos folder. First source the OpenFOAM environmental variables::

    source $HOME/OpenFOAM/OpenFOAM-v1812/etc/bashrc
    
   Then run::
  
    ./Allwmake
    
   Next, go to $HOME/repos/dafoam/python/reg_tests, download `input.tar.gz <https://github.com/mdolab/dafoam/raw/master/python/reg_tests/input.tar.gz>`_ and untar it. Finally, run the regression test there::
  
    python run_reg_tests.py
    
   The regression tests should take less than 30 minutes. You should see something like::
   
    dafoam buoyantBoussinesqSimpleDAFoam: Success!
    dafoam buoyantSimpleDAFoam: Success!
    dafoam calcDeltaVolPointMat: Success!
    dafoam rhoSimpleCDAFoam: Success!
    dafoam rhoSimpleDAFoam: Success!
    dafoam simpleDAFoam: Success!
    dafoam simpleTDAFoam: Success!
    dafoam solidDisplacementDAFoam: Success!
    dafoam turboDAFoam: Success!
  
   You should see the first "Success" in less than 5 minute. If any of these tests fails or they take more than 30 minutes, check the error in the generated dafoam_reg_* files. Make sure all the tests pass before running DAFoam. **NOTE:** The regression tests verify the latest version of DAFoam on Github. However, we use specific old versions for DAFoam's dependencies (e.g., pyGeo, IDWarp, see the following). We recommend using these old versions for the dependencies, which have been routinely tested and verified on Travis::

    Modules              Updated Date    Commit ID 
    BaseClasses          Apr 17, 2019    298ac94b42ef9b057af4f64b329ca81e730fd79e
    IDWarp               Apr 17, 2019    0149681f5fb1edbf86f10d4b038c2cc4c88f68a4
    pyGeo                Apr 17, 2019    90f4b90f98f6f6901f9a724dba2376e7c3c70f4d
    pySpline             Apr 17, 2019    30f2340d721d63ea279851c98fd2a03277ea609e
    pyOptSparse          May 17, 2019    6bc17292b9a569300d1f1e44f378f1a452dccc60
    multipoint           May 09, 2019    68188870d6bb2efcf4e4bfb553757a42fbd68d21
    OpenFOAMMeshReader   Jun 18, 2019    32358d1772737bf6063fe102a646cb18b4fcc1d5

|

In summary, here is the folder structures for all the installed packages::
   
  $HOME
    - OpenFOAM
      - OpenFOAM-v1812
      - ThirdParty-v1812
    - packages
      - anaconda2
      - cgnslib_3.2.1
      - mpi4py-1.3.1
      - petsc-3.6.4
      - petsc4py-3.6.0
    - repos
      - baseclasses
      - cgnsutilities
      - dafoam
      - idwarp
      - multipoint
      - openfoammeshreader
      - pygeo
      - pyhyp
      - pyoptsparse
      - pyspline

Here is the DAFoam related environmental variable setup that should appear in your bashrc file::

  # PETSC
  export PETSC_DIR=$HOME/packages/petsc-3.6.4
  export PETSC_ARCH=real-opt
  export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
  export PATH=$PETSC_DIR/$PETSC_ARCH/include:$PATH
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PETSC_DIR/$PETSC_ARCH/lib
  export PETSC_LIB=$PETSC_DIR/$PETSC_ARCH/lib
  
  # cgns lib
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/packages/cgnslib_3.2.1/src

  # cgns utlis
  export PATH=$PATH:$HOME/repos/cgnsutilities/bin

  # Python path
  export PYTHONPATH=$PYTHONPATH:$HOME/repos

  # Anaconda2
  export PATH="$HOME/packages/anaconda2/bin:$PATH"



  