# ASU RC Software, Python, Containers, and Performance Guidance

Read this before installing packages, creating environments, compiling, using
containers, or tuning performance. The rules in
[`ASU_RC_AGENTS.md`](../ASU_RC_AGENTS.md) still apply.

Current references: [Available software](https://docs.rc.asu.edu/available-software),
[Building software](https://docs.rc.asu.edu/building-software),
[Mamba](https://docs.rc.asu.edu/mamba),
[Python common issues](https://docs.rc.asu.edu/python-common-issues),
[Apptainer](https://docs.rc.asu.edu/apptainer).

## Order of preference

1. An existing Lmod module (`module spider <name>`, `module show <mod>`)
2. An existing project or user environment
3. A new mamba environment in user-controlled storage
4. A user-local source build, inside a `lightwork` or compute job
5. An Apptainer container (`apptainer` runs on compute nodes only)

Voyager `search_software` searches the module catalog without a login, but
it sees **modulefiles only**. Software installed as conda environments or
containers will not appear. If it returns nothing, confirm with
`module spider` before telling the user the software is unavailable. Lmod
also has two naming styles (`gcc/12` versus `gcc-12`); try both.

Lmod startup can add several seconds to a login shell. Account for that when
diagnosing apparent hangs.

## Python

```bash
interactive -p lightwork           # never build environments on a login node
module load mamba/latest
mamba create -n myproj -c conda-forge python=3.12 numpy
source activate myproj             # or: mamba activate myproj
```

- **Never install into `base`.** If the prompt shows `(base)`, deactivate it
  first.
- Do not run `conda init`. It injects code into `~/.bashrc` that breaks later
  sessions. If that has already happened, `remove_conda_from_bashrc` undoes
  it.
- Prefer mamba packages from `conda-forge`. Use `pip` only inside an active
  environment, and never `pip install --user`. Stray `~/.local/lib/python3*`
  directories are a common cause of broken environments.
- Do not install packages from inside a Jupyter notebook. Build the
  environment in a terminal and register it for Jupyter as described in
  [the Jupyter guide](https://docs.rc.asu.edu/jupyter-kernels).
- Prefer `conda-forge` over Anaconda's `defaults` channels unless the user has
  checked that Anaconda's terms of service allow their use.
- In job scripts, set the interpreter explicitly. Do not rely on the
  submitting shell's state:

  ```bash
  module purge
  module load mamba/latest
  source activate myproj
  srun python run.py
  ```

- Record the environment for reproducibility with
  `mamba env export > environment.yml`. Review the file for machine paths
  and secrets before committing it.

## GPUs and AI frameworks

1. Identify the framework, Python, and CUDA versions the code needs.
2. Inside an allocation on the target GPU type, check `nvidia-smi`. Gaudi
   nodes are different: they use `hl-smi` inside the Habana image.
3. Choose a framework build compatible with that driver. The newest build is
   not necessarily the right one.
4. Inside the job, verify that the framework sees exactly the devices it was
   assigned.

Pull large images (vLLM, Habana, NGC) into `/scratch/$USER` from within a
job, never on a login node. For LLM inference, the RC LLM API
([docs](https://docs.rc.asu.edu/ai/api)) usually beats starting a private
model server. Use a private server only when the hosted models do not fit the
task.

## Install boundaries

- No `sudo`, no system-wide installs, and no replacing system libraries.
- Do not modify shared or group environments without the owners' agreement.
- Do not run unreviewed remote installers (`curl … | sh`). Download the
  script, read it, and then decide.
- Do not assume compute nodes can reach the internet. Download on a login
  node (small files) or in a `lightwork` job, before the main job starts.
- Run heavy compiles in Slurm, not on login nodes.

## Performance

- Correctness and reproducibility come first. When performance matters,
  measure a representative workload, find the actual bottleneck, and make a
  targeted change. Do not claim code is optimized without measuring it.
- Improve the algorithm, data structures, I/O, and library choices before
  adding parallelism. Do not parallelize just because cores are available.
- In Python, vectorize or call compiled libraries (NumPy, SciPy, PyTorch, JAX,
  Numba, Polars) instead of large interpreted loops.
- Match thread counts to the allocation and avoid nested oversubscription.
  Derive the values from Slurm rather than hard-coding them:

  ```bash
  export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK:-1}
  export OPENBLAS_NUM_THREADS=$OMP_NUM_THREADS MKL_NUM_THREADS=$OMP_NUM_THREADS
  ```

- Use MPI from a supported module for multi-node work, and benchmark scaling
  before requesting large allocations.
