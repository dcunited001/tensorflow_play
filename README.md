TensorFlow
==========

Just getting up to speed with tensorflow. For starters, I need a place
to host a Dockerfile that's been slightly modified.  This is so I can
customize the build to include CUDA support on a late 2013 MacbookPro,
which is only 3.0 compute capable.

## Building & Running in Docker

### build

```
## hmmm, is this right?
docker build .
```

### run

```
./docker_run_gpu.sh tf/tf
```

trying to run the mnist convolutional network example with

```
python -m tensorflow.models.image.mnist.convolutional
```

and i get `CUDA_ERROR_NO_DEVICE`, so my docker container can't find my gpu.
it's probably because i installed a new version of the CUDA drivers, but
never restarted.  so my host OS can't find the GPU.

```
I tensorflow/core/common_runtime/local_device.cc:40] Local device intra op parallelism threads: 8
E tensorflow/stream_executor/cuda/cuda_driver.cc:481] failed call to cuInit: CUDA_ERROR_NO_DEVICE
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:114] retrieving CUDA diagnostic information for host: 9858e37d8b3b
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:121] hostname: 9858e37d8b3b
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:146] libcuda reported version is: Not found: was unable to find libcuda.so DSO loaded into this program
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:257] driver version file contents: """NVRM version: NVIDIA UNIX x86_64 Kernel Module  352.63  Sat Nov  7 21:25:42 PST 2015
GCC version:  gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04) 
"""
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:150] kernel reported version is: 352.63
I tensorflow/core/common_runtime/gpu/gpu_init.cc:127] DMA: 
I tensorflow/core/common_runtime/direct_session.cc:58] Direct session inter op parallelism threads: 8
```

and reboot linux, but i get `CUDA_ERROR_UNKNOWN`.  this means my GPU's been found, but
there's a problem running code with it.  that's probably because my CUDA driver is 7.5,
where I need 7.0.  I hope it's not because my macbook pro graphics card only
supports nvidia compute capability 3.0 (unsupported) whereas tensorflow
only officially supports 3.5 or higher

```
I tensorflow/core/common_runtime/local_device.cc:40] Local device intra op parallelism threads: 8
modprobe: ERROR: ../libkmod/libkmod.c:556 kmod_search_moddep() could not open moddep file '/lib/modules/3.19.0-32-generic/modules.dep.bin'
E tensorflow/stream_executor/cuda/cuda_driver.cc:481] failed call to cuInit: CUDA_ERROR_UNKNOWN
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:114] retrieving CUDA diagnostic information for host: 24b008aee65f
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:121] hostname: 24b008aee65f
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:146] libcuda reported version is: Not found: was unable to find libcuda.so DSO loaded into this program
I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:257] driver version file contents: """NVRM version: NVIDIA UNIX x86_64 Kernel Module  352.68  Tue Dec  1 17:24:11 PST 2015
GCC version:  gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04)
```

i think it's happening bc i created a separate user to work with docker.
going to add `$LD_LIBRARY_PATH` and other shell vars to that user's bashrc and rebuild.

... apparently that didn't help. still getting the same thing.


`sudo modprobe nvidia` doesn't list anything when run from the host os.  when run from container, i get:

```
modprobe: ERROR: ../libkmod/libkmod.c:556 kmod_search_moddep() could not open moddep file '/lib/modules/3.19.0-32-generic/modules.dep.bin'
```

i checked whether `TF_CUDA_COMPUTE_CAPABILITIES` passed to ./configure by dockerfile makes a difference in the build.  however, it seems to build ok.

but now i'm thinking that the `modprobe` command shouldn't be failing
from the container.



 I also needed to add my docker user to sudo group, so I did that.  but then `/lib/modules` wasn't mapped in the container, so i fixed that with the following addition to my run script

```
export LIB_MODULES=$(\uname -r | xargs -I{} echo '-v /lib/modules/{}:/lib/modules/{}')

# ....

docker run -it $CUDA_SO $LIB_MODULES $DEVICES "$@"
```

but now, when i run `sudo modprobe nvidia` i'm getting 'modprobe: FATAL: Module nvidia not found.'

## Building and Running on Host

mostly referencing dockerfile. this skips a lot of steps btw, not a great reference

```
sudo apt-get update && sudo apt-get install -y \
        build-essential \
        curl \
        git \
        libfreetype6-dev \
        libpng12-dev \
        libzmq3-dev \
        pkg-config \
        python-dev \
        python-numpy \
        python-pip \
        software-properties-common \
        swig \
        zip \
        zlib1g-dev

pip install ipykernel jupyter matplotlib

#also need wheel
pip install wheel

python -m ipykernel.kernelspec

# already have java 8 installed

# already have bazel installed
export BAZELRC=/home/dc/.bazelrc

# tensorflow build args
export CUDA_TOOLKIT_PATH=/usr/local/cuda
export CUDNN_INSTALL_PATH=/usr/local/cuda
export TF_NEED_CUDA=1
export CUDA_PATH=/usr/local/cuda

export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/lib/x86_64-linux-gnu

# ran into problems because i didn't clone with --recursive
# - and didn't pull the protobuf submodule
git clone --recursive https://github.com/tensorflow/tensorflow.git

TF_CUDA_COMPUTE_CAPABILITIES=3.0 TF_UNOFFICIAL_SETTING=1 ./configure && \
    bazel build -c opt --config=cuda tensorflow/tools/pip_package:build_pip_package && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/pip && \
    pip install --upgrade /tmp/pip/tensorflow-*.whl
```

successfully built with python3, but got metaclass errors.

then i built it again, with python2.  but got metaclass errors.

i've built this shit like 14 times today and it takes 30 minutes every
time at 100% x 8 cores.  bazel must be doing some badass optimizations.

turns out [this issue](https://github.com/tensorflow/tensorflow/issues/487)
solve my problems.




