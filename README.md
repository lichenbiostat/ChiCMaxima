# ChiCMaxima

ChiCMaxima pipeline for analyzing  and identificantion of chromation loops in CHi-C promoters data.
Current release: ChiCMaxima 0.9


Installation

Check the Releases section of github for current executable binaries. If a binary isn't available for your system, you can build from source as described below.

Input format

The current version uses ibed matrices with 11 columns in which the coordinates of both interacting regions plus the number of reads.
Format 1: Gzipped matrices: if you have an n-by-n matrix, you can encode it as a text file that contains n tab-separated numbers on each line. For example:

100	100	100	0	0	0	0	0	0	0
100	100	100	0	0	0	0	0	0	0
100	100	100	0	0	0	0	0	0	0
0	0	0	100	100	100	0	0	0	0
0	0	0	100	100	100	0	0	0	0
0	0	0	100	100	100	0	0	0	0
0	0	0	0	0	0	0	0	0	0
0	0	0	0	0	0	0	100	100	100
0	0	0	0	0	0	0	100	100	100
0	0	0	0	0	0	0	100	100	100

Then you gzip this text file. This gzipped file can then be provided to Armatus via the -i flag.

Format 2: The format of Rao et al. domains (see first section above). You must use the -R option.

Format 3: Sparse matrix format from any dataset (3 column text file). You must use the -S option.
Output Format

Armatus will produce files with 3 columns that look like this:

N/A     0       49
N/A     50      99
N/A     100     149

The first column is the chromosome number/name (given with -c) or "N/A" if none is given. The next two columns give the start and ending indices of fragments in a domain. The indices are INCLUSIVE. Each line represents a domain, so the first line above indicates a domain that spans from someplace in fragment 0 to someplace in fragment 49.

If the -r resolution parameter is given, then the above numbers will be scaled by this resolution in the following way:

output_start = start_fragment * resolution
output_end = (end_fragment+1)*resolution - 1

That is: the range will be the nucleotide range that spans the fragments in the domain.
Usage

You can get a list of Armatus' command line arguments by passing the --help parameter. The current arguments are:

-R [ --parseRaoFormat ]               Parse the Rao data format instead of 
                                      Dixon et al. (Provide, for example, 
                                      GM12878_combined/5kb_resolution_intrach
                                      romosomal/chr1/MAPQGE30/chr1_5kb,  and 
                                      KR normalization is used.
-N [ --noNormalization ]              No normalization (Rao et al. format)
-S [ --parseSparseFormat ]            Parse sparse matrix format (input a 3
                                      column text file)
-g [ --gammaMax ] arg                 gamma-max (highest resolution to 
                                      generate domains)
-j [ --justGammaMax ]                 Just obtain domains at the maximum 
                                      Gamma
-h [ --help ]                         produce help message
-i [ --input ] arg                    input matrix file
-k [ --topK ] arg (=1)                Compute the top k optimal solutions
-m [ --outputMultiscale ]             Output multiscale domains to files as 
                                      well
-r [ --resolution ] arg (=1)          Resolution of data
-c [ --chromosome ] arg (=N/A)        Chromosome
-n [ --minMeanSamples ] arg (=100)    Minimum required number of samples to 
                                      compute a mean
-o [ --output ] arg                   output filename prefix
-s [ --stepSize ] arg (=0.050000000000000003)
                                      Step size to increment resolution 
                                      parameter

WARNING

We have noticed that occassionally, the order of the arguments passed can result in a memory bug that is yet unexplained since we are using the Boost argument parser. We are working to resolve this issue. If you experience this issue, place the "-m" argument before the "-k" argument.
Example Run

The main inputs into Armatus are the matrix file (in the format of Dixon et al.: http://chromosome.sdsc.edu/mouse/hi-c/download.html, Rao et al, or any sparse matrix formatted text file) and the gammaMax parameter which determines the highest resolution at which domains are to be generated. Note: we recently noticed that the format of the matrices of Dixon et al. in the link above has changed. An example run on chromosome 1 of a human fibroblast:

time armatus -i IMR90/40kb/combined/chr1.nij.comb.40kb.matrix.gz -g .5 -o test -m

Multiresoultion ensemble will be written to files
Reading input from IMR90/40kb/combined/chr1.nij.comb.40kb.matrix.gz.
chr1 at resolution 40000bp
line 1000
line 2000
line 3000
line 4000
line 5000
line 6000
MatrixParser read matrix of size: 6182 x 6182
gamma=0
gamma=0.05
gamma=0.1
gamma=0.15
gamma=0.2
gamma=0.25
gamma=0.3
gamma=0.35
gamma=0.4
gamma=0.45
gamma=0.5
Writing consensus domains to: test.consensus.txt
Writing multiscale domains

real    1m35.923s
user    1m34.461s
sys 0m1.401s

The first -i parameter is a gzipped HiC matrix for chromosome 1 as obtained from Dixon et al. the second -g parameter is the maximum gamma at which to sample at. Output files are all written with a prefix test.

Other options allow for sampling multiple near-optimal solutions and considering finer levels of step sizes. These are 'idealized' parameters in the sense that ideally we would sample as many resolutions as possible and consider all solutions that are reasonably close to the optimal solution.

We have also added a simple example in the "examples/" directory for your convenience.
Building from Source

Dependencies:

    C++11
    Boost (Graph, Test, ublas, program options, system)

Note: Although this should be conceptually easy, properly installing these dependencies can be tricky since on a system like OS X you can choose to use different compilers and standard libraries. Please make sure that boost is installed with the same standard libary and compiler that you are using to compile Armatus. For this reason, binaries of Armatus will also be released. Even with the binary, you need to install and compile boost.

Here is an example of how to compile boost with Clang++ and use Clang++ to compile Armatus on OS X:

Boost-specific details:

./bootstrap --prefix=$HOME/boost
./b2 clean
./b2 install toolset=clang cxxflags="-stdlib=libc++" linkflags="-stdlib=libc++"

Using CMake to compile Armatus, starting with the root of the git repository as the working directory:

mkdir build
cd build
cmake -DCMAKE_CXX_COMPILER=clang++ -DBOOST_ROOT=$HOME/boost -DBoost_NO_SYSTEM_PATHS=true ..
make
export DYLD_FALLBACK_LIBRARY_PATH=$HOME/boost/lib

Make sure you substitute $HOME/boost with the installation path you desire. Simpler build processes often work as well. For example, if you have boost installed in a standard place, and a standard compiler in a standard place, then it might be sufficient to do:

mkdir build
cd build
cmake ..
make

YMMV.

Here is one-liner to compile Armatus with G++ given you have the proper dependencies:

g++ -std=c++11 -w -L /opt/local/lib/ -I include/ -I /opt/local/include/ -O3 -o binaries/armatus-linux-x64 src/*.cpp -lboost_iostreams -lboost_program_options -lboost_system

Note that this assumes you have Boost installed in "/opt/local".
Components

    armatus executable: (Boost Program Options): Armatus.cpp

ArmatusUtil.{cpp,hpp}:

    parseMatrix(File): 3C matrix parser (Dixon et al. format)
    outputDomains(set of domains)
    consensusDomains(set of domains)
    multiscaleDomains(gammaMax, stepSize, k)

Data structures {cpp,hpp}:

    ArmatusParams(Matrix, gamma): parameters to DP and pre-computable quantities such as the quality function
    ArmatusDAG(Params): encodes structure of dynamic program
        BackPointers and SubProblem classes: see below
        build(): Build the DAG for the dynamic program (Boost Graph)
        computeTopK(k): At every node, store 'SubProblem': k 3-tuples: (edge, child solution, score of kth best solution for this subproblem)
        extractTopK(k): Returns a set of domains
    IntervalScheduling
        WeighedInterval
        IntervalScheduler
            previousDisjointInterval()
            computeScheduling()
            extractIntervals()

Testing:

    Boost test

Pipeline:

    Code and data links to generate results in application note
