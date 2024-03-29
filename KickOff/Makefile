# This Makefile is brutally simple, not very sophisticated,
# and not at all up to my normal standards...
# ...Be grateful, dear reader!
# (Signed, Prof. R. Moore, h_da, fbi )

# --------------------------------------------------------------------------

# Choose a CXX and a C Compiler!
# Ether gnu compiler chain (gcc):
# CXX = g++
# CC = gcc
# Or clang / LLVM:
CXX = clang++
CC = clang
# Or whatever is the default on your system:
# CXX = c++
# CC = cc

export OMPI_CC=$(CC)
export OMPI_CXX=$(CXX)

# Depending on your OS and local installation of clang, openmp, MPI, etc.,
# you MIGHT need to uncomment one or more of the following lines:
# CPPFLAGS += -std=c++11 -I/user/local/include/
# LDFLAGS  += -std=c++11 -L/user/local/lib/
#         The next two lines are only needed for Mac OS (if at all):
# CPPFLAGS += -I/opt/homebrew/opt/llvm/include/
# LDFLAGS  += -L/opt/homebrew/opt/llvm/lib/

# This should work on all OS's (
CPPFLAGS +=  --std=c++17 -O3 -Wall
LDFLAGS  +=  --std=c++17 -O3 -Wall

# This makefile makes the following programs
TARGETS = mpi-pi mpi-pi++ cpp11-pi openMP-pi 

all: $(TARGETS)

clean:
	-rm -f *.o $(TARGETS)

# NOTE : It is possible to tell mpicc which c (or c++) compiler to use, 
# but the method depends on whether you are using OpenMPI or MPICH.
# For MPICH, use 
#	mpicc -cc="$(CC)" -O3 ... 
# For OpenMPI, see "man mpicc" (environment variables, toward the bottom of the page)

# Help with Makefile "automatic variables":
# (https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html#Automatic-Variables)
# ---------------------------------------------
# $@ = The filename representing the target.
# $< = The filename of the first prerequisite.
# $(*F) = The stem of the filename of the target (i.e. without .o, .cpp...)
# $^ = The names of all the prerequisites, with spaces between them.

show:
	mpicc --showme:command
	mpic++ --showme:command

mpi-pi.o: mpi-pi.c
	mpicc -O3 -c $<

mpi-pi: mpi-pi.o
	mpicc -O3 -o $@ $^
	
mpi-pi++.o: mpi-pi++.cpp	
	mpic++ $(CPPFLAGS) -c $<

mpi-pi++: mpi-pi++.o
	mpic++ $(LDFLAGS) -o $@ $^
	
cpp11-pi.o: cpp11-pi.cpp
	$(CXX) $(CPPFLAGS) -pthread -c $<

cpp11-pi: cpp11-pi.o
	$(CXX) $(LDFLAGS) -pthread -o $@ $^

openMP-pi.o: openMP-pi.cpp
	$(CXX) $(CPPFLAGS) -fopenmp -c $<

openMP-pi: openMP-pi.o
	$(CXX) $(LDFLAGS) -fopenmp -o $@ $^

NUM_THREADS = $(shell grep -E 'core id' /proc/cpuinfo | sort | uniq | wc -l)
# Get the number of physical cores to not need to edit after every commit/push/pull

result.csv: runner.sh $(TARGETS)
	bash runner.sh

benchmark: result.csv

tests: all
ifeq (, $(shell which srun)) # When srun is not available, assume we want to run locally
	echo "\nRunning cpp11-pi without slurm...";
	./cpp11-pi $(NUM_THREADS) 500
	echo "\nRunning openMP-pi without slurm...";
	./openMP-pi $(NUM_THREADS) 500
	echo "\nRunning mpi-pi++ without slurm...";
	mpiexec --mca btl tcp,self -n $(NUM_THREADS) ./mpi-pi++ 500
	echo "\nRunning mpi-pi without slurm...";
	mpiexec --mca btl tcp,self -n $(NUM_THREADS) ./mpi-pi 500
else # if srun is available, use it -- e.g. at the GSI!
	srun -p debug -c $(NUM_THREADS) -- ./cpp11-pi $(NUM_THREADS) 500
	srun -p debug -c $(NUM_THREADS) -- ./openMP-pi $(NUM_THREADS) 500
	salloc -p debug --nodes=$(NUM_THREADS) --ntasks-per-node=1 -c 1 -- srun -- ./mpi-pi 500
	salloc -p debug --nodes=$(NUM_THREADS) --ntasks-per-node=1 -c 1 -- srun -- ./mpi-pi++ 500
endif

# Note: The test whether srun is available (above) was provided by Jakob Schrettenbrunner.
# Many thanks!  However, it might not work with versions of make other than gnu make.
# If it doesn't work, either install gnu make, 
# or simply comment out the lines you do not want to use.

# Note on "--mca btl tcp,self" (with mpiexec, see above)
# See https://www.open-mpi.org/faq/?category=tcp
# (This helps on macOS, may be unnecessary elsewhere, or even problematic!)

# Note on "mpiexec" vs. "mpirun" (above)
# They are often one and the sam thing -- but not everywhere;
# See https://stackoverflow.com/questions/25287981/mpiexec-vs-mpirun
