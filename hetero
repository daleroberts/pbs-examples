#!/bin/bash
#PBS -P v10
#PBS -q normal
#PBS -l walltime=00:01:00,mem=200MB
#PBS -l ncpus=32
#PBS -j oe

export RANKFILE=~/rankfile

module load python/2.7.6
module load openmpi/1.8

python --version | head -n1
mpirun --version | head -n1
python -c "import numpy; print('numpy' + numpy.__version__)"

# heterogeneous allocation

python - << EOF
import os
uniqs = set()
nodefilename = os.getenv('PBS_NODEFILE')
rankfilename = os.getenv('RANKFILE')
pernode = int(os.getenv('PBS_NCPUS')) / 8
with open(nodefilename, 'r') as nodefile:
  nodes = [n.strip() for n in nodefile.readlines()]
  nodes = [n for n in nodes if n not in uniqs and not uniqs.add(n)]
  with open(rankfilename, 'w') as rankfile:
    rankfile.write('rank 0=%s slot=0:0\n' % nodes[0])
    rankfile.write('rank 1=%s slot=0:1-%i\n' % (nodes[0], pernode - 1))
    for i, node in enumerate(nodes[1:]):
      rankfile.write('rank %i=%s slot=0:0-%i\n' % (i+2, node, pernode - 1))
EOF

BIN='python -e "import pypar as pp; print pp.rank(), pp.size(); pp.finalize()"'
NP=$(expr $PBS_NPCPUS / 8 - 1)

mpirun -report-bindings -tag-output -display-map -rf ${RANKFILE} \
	-np 1 -x OMP_NUM_THREADS=1 $BIN : \
	-np 1 -x OMP_NUM_THREADS=7 $BIN : \
	-np $NP -x OMP_NUM_THREADS=8 $BIN 

rm $RANKFILE
