TensorFlow
==========

Just getting up to speed with tensorflow. For starters, I need a place
to host a Dockerfile that's been slightly modified.  This is so I can
customize the build to include CUDA support on a late 2013 MacbookPro,
which is only 3.0 compute capable.

build

```
## hmmm, is this right?
docker build .
```

run

```
./docker_run_gpu.sh tf/tf
```



### after first build

trying to run the mnist convolutional network example with

```
python -m tensorflow.models.image.mnist.convolutional
```

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
