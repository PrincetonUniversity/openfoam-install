**Installing OpenFOAM on Princeton University's Clusters**

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgcb84ce4">1. Create Directories</a></li>
<li><a href="#org101563b">2. Checking Requirements and Downloading Software</a>
<ul>
<li><a href="#orgd7774bf">2.1. Software Requirements</a></li>
<li><a href="#orgc16421b">2.2. Downloading the OpenFOAM and ThirdParty Source Code</a></li>
</ul>
</li>
<li><a href="#orga328b6b">3. Setting OpenFOAM Environment Variables</a></li>
<li><a href="#org161fc74">4. Compilation</a></li>
</ul>
</div>
</div>


<a id="orgcb84ce4"></a>

# Create Directories

OpenFOAM installation requires at least 30 GB of disk space. In this document we will create a directory called `OPENFOAM_WORKDIR` and assume that all the packages will be downloaded and compiled there. For example, on `adroit`,
you could create this directory as `/scratch/network/$USER/OpenFOAM`:

    $ mkdir /scratch/network/$USER/OpenFOAM
    $ export OPENFOAM_WORKDIR=/scratch/network/$USER/OpenFOAM


<a id="org101563b"></a>

# Checking Requirements and Downloading Software

Now let's download OpenFOAM.
Here, we basically follow the instructions in [OpenFOAM Source Repository](https://openfoam.org/download/source/).

<a id="orgd7774bf"></a>

## Software Requirements

The software requirements are listed on the page [OpenFOAM Repo: 1. Software for Compilation](https://openfoam.org/download/source/software-for-compilation/).

Let's check that we have the necessary tools and libraries:

-   `gcc`, `flex`, `cmake` and `git` are on perseus by default and are the right versions,
-   to access `qtmake`, you need to add:
    
        export QMAKE_PATH=/usr/lib64/qt5/bin
        export PATH=$QMAKE_PATH:$PATH

    to your file `.bashrc`.
-   to access `OpenMPI`, you need to add:
    
        module load openmpi/gcc/4.1.0
    
    to your `.bashrc`

These are the only steps that you need to take to satisfy the software
dependencies.


<a id="orgc16421b"></a>

## Downloading the OpenFOAM and ThirdParty Source Code

The page [OpenFOAM Repo: 2. Downloading Source Code](https://openfoam.org/download/source/downloading-source-code/) suggests cloning the git repository,
but here we will only download the latest releases of OpenFOAM and ThirdParty.
This is done through following steps:

    $ cd $OPENFOAM_WORKDIR
	$ git clone git@github.com:OpenFOAM/OpenFOAM-9.git
    $ git clone git@github.com:OpenFOAM/ThirdParty-9.git

In addition to the software that comes within the `ThirdParty` repository, we need
to install two additional packages: `CGAL` and `boost`, here is how to do it:

    $ cd $OPENFOAM_WORKDIR/ThirdParty-9.x
    $ wget https://github.com/CGAL/cgal/archive/refs/tags/v5.4.1.tar.gz
    $ tar xf v5.4.1.tar.gz
    $ rm v5.4.1.tar.gz
	$ wget wget https://sourceforge.net/projects/boost/files/boost/1.79.0/boost_1_79_0.tar.bz2
    $ tar xf boost_1_79_0.tar.bz2
    $ rm boost_1_79_0.tar.bz2

Now we need to make some changes to tell OpenFOAM were to find the source for `CGAL` and 
`root`. This is done in the file `OpenFOAM-9/etc/config.sh/CGAL` and can be performed
with two `sed` commands:

    $ cd OPENFOAM_WORKDIR
    $ sed -i -e 's/\(boost_version=\)boost-system/\1boost_1_79_0/' OpenFOAM-9/etc/config.sh/CGAL
    $ sed -i -e 's/\(cgal_version=\)cgal-system/\1cgal-5.4.1/' OpenFOAM-9/etc/config.sh/CGAL


<a id="orga328b6b"></a>

# Setting OpenFOAM Environment Variables

OpenFOAM requires some environment variables to be defined to compile and run.
There is a script that comes with OpenFOAM sources that sets up the appropriate
environment variables.
All we have to do is to add this line to our `.bashrc`:

    source $OPENFOAM_WORKDIR/OpenFOAM-9/etc/bashrc

where you need to replace `$OPENFOAM_WORKDIR` with its value and then source the `.bashrc` for the changes to take effect:

    $ source ~/.bashrc


<a id="org161fc74"></a>

# Compilation

Now we are finally ready to compile OpenFOAM and the ThirParty software:

    $ cd $OPENFOAM_WORKDIR/OpenFOAM-9
    $ ./Allwmake -j 4 2>&1 | tee make.log

where the options for `Allwmake` mean:

-   `-j 4` compile in parallel on 4 cores. We should limit ourselves
    to 4 cores in order to leave enough resources for other users.
-   `2>&1` redirects standard error to standard output <sup><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>.
-   `| tee make.log` sends standard output to the `tee` Linux command, which
    does two things: 
    1.  print its input to the screen,
    2.  copy its input to the file `make.log`.

This compilation operation will take a long time (possibly a couple hours). So it
would be best to run it within a terminal multiplexer like [tmux](https://github.com/tmux/tmux/wiki), so that it will
keep running even if we loose our connection to perseus.


<a id="org822e47f"></a>


# Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> [Standard streams - Wikipedia](https://en.wikipedia.org/wiki/Standard_streams)
