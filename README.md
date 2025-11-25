## Steps to run script on TIR Training cluster

Launch a slurm deployment. Use Nemo Image. Add a PFS at mount path /pfs. 

1. Change perms on pfs 
```
chown -R admin /pfs
```
2. Set HF home to write files to pfs. This will allow us to download tokenizer just once 

```
$ export HF_HOME=/pfs/hf
$ huggingface-cli login --token <ADD-YOUR-HF-TOKEN-HERE>
$ huggingface-cli download --repo-type model meta-llama/Llama-3.1-8B
```

We will use this model as tokenizer. Confirm it is available in `/pfs/hf/hub/models--meta-llama--Llama-3.1-8B/snapshots/d04e592bb4f6aa9cfee91e2e20afa771667e1d4b`

3. Clone this repo
```
$ cd /pfs
$ git clone https://github.com/mindhash/dgxc-bench
$ mv dgxc-bench e2e_perf
$ cd e2e_perf
```

4. Get the list of infiniband ports:
```
$ sudo apt update
$ sudo apt install ibverbs-utils libibverbs-dev libibumad3 libibumad-dev librdmacm-dev rdmacm-utils infiniband-diags ibverbs-utils
$ ibv_devinfo 
# From the ibstat output, list out the infiniband ports
$ export NCCL_IB_HCA=mlx5_0,mlx5_9,mlx5_6,mlx5_5,mlx5_4,mlx5_3,mlx5_11,mlx5_10
```  
5. Run the following
Note: You may have to edit `configure.sh` (from this repo) to correct IB ports if they are different from previous step.

```
$ export NEMORUN_HOME=/pfs/nemorun_home
$ export HOME=/home/admin
$ export NUM_NODES=4
$ NEMO_LOG_TRAIN_LOSS=1 NEMO_LOG_MEMORY_USAGE=1 STAGE_PATH=/pfs/e2e_perf/workload_8b   DTYPE=bf16 MODEL_SIZE=8b sbatch -A ${SLURM_ACCOUNT} -p ${SLURM_PARTITION} -N ${NUM_NODES} ./launch.sh

```
