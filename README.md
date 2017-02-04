# Setting up TensorFlow-GPU on Mac

Installing GPU-enabled TensorFlow on Mac is a torture. It takes hours to find the right tweaks. While there are only few people using NVIDIA GPUs on their mac because of Apple's recent adoption of AMD chips, this instruction will still benefit the users of Hackintosh or old Mac Pros. I'm not sure whether this will work on eGPUs, if it does, it could be great news for other Mac users with their purchase of eGPU boxes like Razor Core or Akitio Node.

Performance wise, the GPU version took about 3 minutes to run the "deep MNIST for experts" tutorial. 

`https://www.tensorflow.org/tutorials/mnist/pros/`

The cpu version did that in about half an hour. Apparantly, the 10x performance boost is worth the chore. 

      My System:
       i5-4690k
       NVIDIA GeForce GTX 970
       macOS 10.12.3
       python 2.7.12
       TensorFlow 0.12
       Xcode 7.2.1/8.2.1
       cuda 8.0.54, cudnn 5.1, nvidia-367
  
1. Keep two versions of Xcode: Xcode7.3 (7.2 also works) and Xcode8.x

2. Set Xcode directory to Xcode8, update brew, cask, bazel, cuda as Official setup guide
  Change directory of Xcode to the right version (may be somewhere else):
`sudo xcode-select -s /Applications/XCode8.2/Xcode.app/`
  You can type  `gcc --version` to check LLVM version
3. Set Xcode directory to Xcode7 to compile cuda samples to test whether the installation is a success. Don't forget to add cuda path to `~/.bash_profile`
       export CUDA_HOME=/usr/local/cuda
       export DYLD_LIBRARY_PATH="$DYLD_LIBRARY_PATH:$CUDA_HOME/lib"
       export PATH="$CUDA_HOME/bin:$PATH"

    After installing cuda type in
        sudo ln -s /usr/local/cuda/lib/libcuda.dylib /usr/local/cuda/lib/libcuda.1.dylib` 
to create symbolic link so as to prevent segmentation fault when importing TensorFlow later

4. Choose the right distribution of TensorFlow and pip install it. You can try, but most likely, python will fail importing TensorFlow

5. Then try installing TF from source.

        git clone https://github.com/tensorflow/tensorflow
        cd tensorflow
  Now you are in the TensorFlow workspace. Run `./configure` to set directories and versions
  Choose default except for cuda to enable cuda
  Currently, enter cuda version as 8.0 and cudnn version as 5 (though 5.1 in fact)
  Check GPU computability on Nvidia's website:
  
  `https://developer.nvidia.com/cuda-gpus`
  
  According to the official installation guide, when done configuring you should enter
  
  `bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package`
  
  to start building. 
  
  However, most likely it will throw you an error `dyld: Library not loaded: @rpath/libcudart.8.0.dylib`
  To solve the problem see
  
  `https://github.com/JimmyKon/tensorflow_build_issue_fix/tree/master`
  
  You will have to modify the `genrule-setup.sh` in your temp folder following the instructions.
  You can search or find it here:
  
  `/private/var/tmp/.../execroot/tensorflow/external/bazel_tools/tools/genrule/genrule-setup.sh`
  
  After modification, run `./configure` in TensorFlow workspace again and start building with
  
  `bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package`
  
  This will take a long time, enjoy your life. Continue with
  
  `bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg`
  
  `sudo pip install --upgrade --ignore-installed  /tmp/tensorflow_pkg/tensorflow-*.whl`
  
  If you are using something like anaconda, `--ignore-installed` is necessary, `sudo` is related to your pip install
 
  At this time, try importing TensorFlow in your python. If it throws you a `keyerror:...`
  Try installing this version of protobuf provided by Google
  
      sudo pip install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/protobuf-3.1.0-cp27-none-macosx_10_11_x86_64.whl 
      
  I don't know whether there is a proper GPU version, this version is working on my system. If it doesn't work for you, try
        sudo pip install --upgrade protobuf
        
6. If you encountered other errors, check the official setup guide for more information.

Reference

https://www.tensorflow.org/get_started/os_setup

https://srikanthpagadala.github.io/notes/2016/11/07/enable-gpu-support-for-tensorflow-on-macos

https://github.com/JimmyKon/tensorflow_build_issue_fix/tree/master
