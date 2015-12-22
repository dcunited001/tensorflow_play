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

