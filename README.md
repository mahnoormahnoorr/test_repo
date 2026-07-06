module purge
module use /appl/local/laifs/modules
module load lumi-aif-singularity-bindings

export SIF=/appl/local/laifs/containers/lumi-multitorch-u24r70f21m50t210-20260513_121430/lumi-multitorch-full-u24r70f21m50t210-20260513_121430.sif


singularity shell "$SIF"

Singularity> python -m venv optimum-env --system-site-packages
Singularity> source optimum-env/bin/activate
(optimum-env) Singularity> pip install optimum==1.27.0
(optimum-env) Singularity> pip install gptqmodel==4.0.0 --no-build-isolation --cache-dir ./.pip-cache
(optimum-env) Singularity> pip install llmcompressor==0.7.1 --cache-dir ./.pip-cache


#!/bin/bash
#SBATCH --account=project_462001302
#SBATCH --partition=dev-g
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --gpus-per-node=1
#SBATCH --mem=32G
#SBATCH --time=00:30:00
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

# Load the module
module purge
module use /appl/local/laifs/modules
module load lumi-aif-singularity-bindings

export SIF=/appl/local/laifs/containers/lumi-multitorch-u24r70f21m50t210-20260513_121430/lumi-multitorch-full-u24r70f21m50t210-20260513_121430.sif

export CACHE_ROOT=/scratch/project_462001302/mmahnoor/llm-quantization-scripts/GPTQ/hf-cache
mkdir -p $CACHE_ROOT

export HF_HOME=$CACHE_ROOT
export SINGULARITYENV_HF_HOME=$HF_HOME

export HF_DATASETS_CACHE=$CACHE_ROOT/datasets
export SINGULARITYENV_HF_DATASETS_CACHE=$HF_DATASETS_CACHE

export XDG_CACHE_HOME=$CACHE_ROOT/xdg
export SINGULARITYENV_XDG_CACHE_HOME=$XDG_CACHE_HOME

export TORCH_HOME=$CACHE_ROOT/torch
export SINGULARITYENV_TORCH_HOME=$TORCH_HOME

mkdir -p $HF_DATASETS_CACHE $XDG_CACHE_HOME $TORCH_HOME

export HF_TOKEN=xxx
export SINGULARITYENV_HF_TOKEN=$HF_TOKEN


srun singularity exec -B /scratch/${SLURM_JOB_ACCOUNT} --cleanenv "$SIF" \
  bash -lc 'source optimum-env/bin/activate && python3 gptq-config.py'
