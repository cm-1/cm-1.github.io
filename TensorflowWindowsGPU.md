Tensorflow GPU only works on Windows without WSL2 up until a certain older version.

To get it working, I first needed to create a conda env using Python 3.7.

Then, I ran what the Tensorflow Website said to:

```bash
conda install -c conda-forge cudatoolkit=11.2 cudnn=8.1.0
# Anything above 2.10 is not supported on the GPU on Windows Native
python -m pip install "tensorflow<2.11"
# Verify the installation:
python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

Then, to get things working with .ipynb files, I used conda to install ipykernel.

Then I installed pandas and matplotlib with pip.

But then I got this **error:** `libdevice not found at ./libdevice.10.bc`

A few solutions online suggested running `conda install -c anaconda cudatoolkit`; I did this, but I suspect it was unnecessary and just caused more problems down the line with numpy and urllib3 and whatnot that I had to solve below. In any event, it did not solve the problem immediately.

Then, I tried a combination of answers at "[TensorFlow libdevice not found. Why is it not found in the searched path?](https://stackoverflow.com/questions/68614547/tensorflow-libdevice-not-found-why-is-it-not-found-in-the-searched-path)" and "[Can't find libdevice directory `${CUDA_DIR}/nvvm/libdevice. #58681`](https://github.com/tensorflow/tensorflow/issues/58681)" 

First, I found the `libdevice.10.bc` file at `C:\Users\USERNAME\miniconda3\envs\ENVNAME\Library\bin`, created the folder `C:\Users\USERNAME\miniconda3\envs\ENVNAME\Library\bin\nvvm\libdevice`, and **copied** (not cut) the file to that folder. 

That did not do it by itself, and what's worse, at this point, I had realized that my cudatoolkit install had messed up my pandas install… the following commands were tried in order; generally, at least one thing would break with each command, necessitating the one ahead of it, until finally it was no longer one step forward one step back.
```bash
conda uninstall cudatoolkit
conda uninstall numpy-base
python -m pip install ipykernel
python -m pip install numpy==1.21.6 --force-reinstall
```

At this point, pandas worked again, but now tensorflow broke at an earlier point than before… 
This time, my error was `ImportError: urllib3 v2.0 only supports OpenSSL 1.1.1+, currently the 'ssl' module is compiled with LibreSSL 2.8.3`.
Online, `pip install urllib3==1.26.6` was suggested as a solution, but then this produced a different error: `cannot import name 'get_host' from 'urllib3.util.url'`
So instead, based on a lower reply at [https://github.com/invoke-ai/InvokeAI/issues/3358](https://github.com/invoke-ai/InvokeAI/issues/3358), I used:
```bash
pip uninstall urllib3
pip install 'urllib3<2.0'
```

Finally, TensorFlow was breaking at the same point as before, with not finding `libdevice.10.bc`!
So, going back to the respective StackOverflow post and GitHub issue, I tried `conda install -c nvidia cuda-nvcc` as per a later GitHub suggestion, but that didn't do anything. 

But setting `$env:XLA_FLAGS=--xla_gpu_cuda_data_dir=C:\Users\USERNAME\miniconda3\envs\ENVNAME\Library\bin` worked.

It's a bit of a pain to set it each time it runs, but at some point, I can follow the steps at [https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#windows](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#windows) to have it set on each activation.

Then, I got a new error: `Failed to load in-memory CUBIN`. Thankfully, as suggested at [https://github.com/google/jax/issues/18572](https://github.com/google/jax/issues/18572), updating my GPU driver fixed it (and I did not even have to restart my PC, even though the driver installer suggested that I do).

Then, it worked! I confirmed via Task Manager that it's using my GPU to train models. But I suspect a lot off the above steps were unnecessary, and maybe added an extra useless GB of memory. So I may try redoing it without the missteps.