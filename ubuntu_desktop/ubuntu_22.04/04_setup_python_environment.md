# A Guide to Python Environment Setup on Ubuntu 22.04

Managing Python environments can be a frustrating experience. System-level packages conflict with project-specific dependencies, leading to the **dependency hell**. The solution is to use a dedicated environment manager.

This guide will walk you through setting up a fast, clean, and powerful Python development environment on Ubuntu 22.04 using **Miniforge3** and **Mamba**

**Miniforge3** is minimal, community-driven installer for the Conda environment manager. Unlike the full **Anaconda** installer, it is lightweight and defaults to the superior `conda-forge` channel for packages.

**Mamba** is a lightning-fast, drop-in replacement for the `conda` command. It is written in C++ and can resolve complex dependencies orders of magnitude faster than conda, saving you a massive amount of time and frustration. **Miniforge** comes with **Mamba** pre-installed.

You can download the **Miniforge3** by `wget` command:

```sh
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
```

Install **Miniforge3** by running the script:

```sh
bash Miniforge3-$(uname)-$(uname -m).sh
```

Remember to run `conda init` at the end of your installation in your bash to activate the mamba command. When opening new terminal, it automatically activates the `base` environment.

To prevent automatically activating `base` environment, run following command:

```sh
conda config --set auto_activate_base false
```

After running this, when you open a terminal, you will stay in the system environment (no `base` activated).

### Basic `mamba` command

`mamba` is a fast, drop-in replacement for the `conda` command. So the syntax is similar `conda`.

Create a new environment.

```sh
mamba create -n name_env python=version
```
 Activate an environment.

```sh
mamba activate name_env
```

Deactivate an activated environment.
    
```sh
mamba deactivate
```

Install packages.

```sh
mamba install packages
```

Update packages.

```sh
mamba update packages
```

Remove packages.

```sh
mamba remove packages
```

Clean cache.

```sh
mamba clean --all
```


List packages of an environment.

```sh
mamba list
```

List all environments.

```sh
mamba env list
```

Export an environment to `YAML` file.

```sh
mamba env export -n name_env > name_env.yaml
```

Create an environment from imported file.

```sh
mamba env create -f name_env.yaml
```

## Conclusion
You have successfully set up a professional-grade Python development environment on your Ubuntu machine. By using **Miniforge3** and **Mamba**, you've gained a system that is isolated, fast, powerful and reproducible.

