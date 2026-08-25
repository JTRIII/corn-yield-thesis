# Anaconda Environment Setup for a Jupyter Lab/Notebook Project

### To create a Conda environment:
- Use the command `conda create --name <env_name> python=<version>`
- Replace `<env_name>` with the desired name for your environment and `<version>` with the Python version you want to use (e.g., 3.9). 
- Activate the environment with `conda activate <env_name>` 
- Deactivate it with `conda deactivate`

### To delete a Conda environment:
- If the environment you want to delete is currently active, you need to deactivate it first. Use the command `conda deactivate`
- List all environments with `conda env list`
- Use the following command to remove the environment: `conda env remove --name <environment_name>`
- List all environments with `conda env list`


### To create a Conda kernel for Jupyter:
- To list available Conda kernels in Jupyter, use the command `jupyter kernelspec list`
- If you want to see the kernels associated with your active Conda environment, first activate the environment using `conda activate <env_name>`
- Then, you can install *ipykernel* within that environment using `conda install -c conda-forge ipykernel`
- Create a kernel for it with `python -m ipykernel install --user --name=<env_name>`
- After that, the kernel should be visible when you start Jupyter Notebook or JupyterLab

### To delete a Conda kernel for Jupyter:
- First, identify the kernel name using `jupyter kernelspec list` 
- Then, use the `jupyter kernelspec uninstall <kernel_name>` command to remove it. 

#### Here is a link to the [Conda Documentation](https://docs.conda.io/projects/conda/en/stable/index.html).