# Llama.cpp

## Build in workspace -

Open Terminal in /projects/llama-cpp

```bash
mkdir /projects/llama-cpp-build
cd /projects/llama-cpp-build
cmake /projects/llama-cpp -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DLLAMA_CURL=ON -DGGML_CCACHE=OFF -DGGML_NATIVE=OFF
cmake --build /projects/llama-cpp-build --config Release -j -v
cmake --install /projects/llama-cpp-build --prefix ${HOME}/.local
```

```
ramalama pull granite3.2:2b
ramalama pull granite3.2:8b
ramalama pull granite-embedding:30m

llama-server --model ${RAMALAMA_STORE}/models/ollama/granite3.2:2b --host 0.0.0.0 --n-gpu-layers 999 --flash-attn --ctx-size 131072
llama-server --model ${RAMALAMA_STORE}/models/ollama/granite3.2:8b --host 0.0.0.0 --n-gpu-layers 999 --flash-attn --ctx-size 131072
llama-server --model ${RAMALAMA_STORE}/models/ollama/granite-embedding:30m --host 0.0.0.0 --n-gpu-layers 999 --flash-attn --ctx-size 512
```

## Notes for Container Build -

```bash
cat << EOF > /etc/yum.repos.d/oneAPI.repo
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF

dnf install -y g++ cmake git libcurl-devel intel-oneapi-mkl-sycl-devel intel-oneapi-dnnl-devel intel-oneapi-compiler-dpcpp-cpp intel-level-zero oneapi-level-zero oneapi-level-zero-devel intel-compute-runtime

# Build llama.cpp
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
mkdir -p build
cd build
source /opt/intel/oneapi/setvars.sh
cmake .. -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DLLAMA_CURL=ON -DGGML_CCACHE=OFF -DGGML_NATIVE=OFF
cmake --build . --config Release -j -v
cmake --install . --prefix /usr

firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib64:/usr/local/lib/

llama-server --model granite-code:3b --host 0.0.0.0 --n-gpu-layers 999 --flash-attn --ctx-size 32768
```

## Whisper.cpp

```bash
git clone https://github.com/ggerganov/whisper.cpp
cd whisper.cpp
mkdir -p build
cd build
source /opt/intel/oneapi/setvars.sh
cmake .. -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_CCACHE=OFF -DGGML_NATIVE=OFF
cmake --build . --config Release -v

# Notes - Not Necessarily Working...
mkdir models
chmod 777 models
podman run -it --name llama --rm --device /dev/dri -p 8080:8080 -v ${PWD}/models:/models:Z quay.io/cgruver0/llama-cpp-intel-gpu:latest /bin/bash

--offload-new-driver

```bash
tee > /etc/yum.repos.d/oneAPI.repo << EOF
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF

dnf install intel-oneapi-base-toolkit

dnf install -y libgudev-devel g++ cmake git kernel-devel lspci clinfo intel-level-zero oneapi-level-zero oneapi-level-zero-devel intel-igc-devel.x86_64 intel-gmmlib-devel ninja-build  intel-opencl-clang-devel libcurl-devel intel-opencl mesa-dri-drivers mesa-vulkan-drivers mesa-vdpau-drivers mesa-libEGL mesa-libgbm mesa-libGL mesa-libxatracker libvpl-tools libva libva-utils intel-gmmlib intel-ocloc intel-metee intel-metee-devel

lspci -nn |grep  -Ei 'VGA|DISPLAY'
lspci -nn |grep  -i accel

git clone https://github.com/intel/linux-npu-driver.git
cd linux-npu-driver/
git submodule update --init --recursive
cmake -B build -S .
cmake --build build --parallel $(nproc)
cmake --install build --prefix /usr
rmmod intel_vpu
modprobe intel_vpu

git clone https://github.com/intel/compute-runtime.git -b releases/24.52

mkdir compute-runtime/build
cd compute-runtime/build
cmake -DCMAKE_BUILD_TYPE=Release -DNEO_SKIP_UNIT_TESTS=1 ../
make -j`nproc`
make install

git clone https://github.com/ggerganov/llama.cpp.git -b b4502
cd llama.cpp
mkdir -p build
cd build
source /opt/intel/oneapi/setvars.sh
cmake .. -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_SYCL_F16=ON -DLLAMA_CURL=ON -DGGML_CCACHE=OFF 
cmake --build . --config Release -j -v

firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

./bin/llama-server --model granite-code:3b --model-url ollama://granite-code:3b --host 0.0.0.0
```

```
intel-media libmfxgen1 libvpl2 level-zero intel-level-zero-gpu mesa-libxatracker libvpl-tools intel-metrics-discovery intel-metrics-library intel-igc-core intel-igc-cm libmetee intel-gsc

cmake -B build -DGGML_NATIVE=OFF -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DLLAMA_CURL=ON -DGGML_CCACHE=OFF

-DBUILD_SHARED_LIBS=OFF

-DGGML_NATIVE=OFF -DBUILD_SHARED_LIBS=OFF

-DGGML_SYCL_DEVICE_ARCH=mtl_h -DCXX_FLAGS="--offload-new-driver"

-- Installing: /etc/OpenCL/vendors/intel.icd
-- Installing: /usr/local/bin/ocloc-24.52.1
-- Installing: /usr/local/lib64/libocloc.so
-- Installing: /usr/local/include/ocloc_api.h
-- Installing: /usr/local/lib64/intel-opencl/libigdrcl.so
```

## Build for bundling

```bash
cat << EOF > /etc/yum.repos.d/oneAPI.repo
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF

dnf install -y lspci clinfo intel-opencl g++ cmake git tar libcurl-devel intel-oneapi-base-toolkit
git clone https://github.com/ggerganov/llama.cpp.git -b b4523
cd llama.cpp
mkdir -p build
cd build
source /opt/intel/oneapi/setvars.sh
cmake .. -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DLLAMA_CURL=ON -DGGML_CCACHE=OFF -DGGML_NATIVE=ON
cmake --build . --config Release -j -v
cmake --install . --prefix /tmp/llama-cpp
cd /tmp/llama-cpp
tar -cvf llama-cpp-bundle ./
```

Packages - intel-oneapi-runtime-compilers intel-oneapi-mkl-core intel-oneapi-mkl-sycl-blas intel-oneapi-runtime-dnnl

export LD_LIBRARY_PATH=/opt/intel/oneapi/redist/lib:/opt/intel/oneapi/redist/lib/clang/19/lib:/opt/intel/oneapi/redist/opt/compiler/lib

## build with UBI 9 - Does not work yet.

```bash

cat << EOF > /etc/yum.repos.d/oneAPI.repo
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF

dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
dnf install -y lspci clinfo g++ cmake git tar libcurl-devel intel-oneapi-base-toolkit

git clone https://github.com/intel/gmmlib.git
cd gmmlib
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j"$(nproc)"
make install



git clone https://github.com/intel/compute-runtime.git -b releases/24.52

mkdir compute-runtime/build
cd compute-runtime/build
cmake -DCMAKE_BUILD_TYPE=Release -DNEO_SKIP_UNIT_TESTS=1 ../
make -j`nproc`
make install

git clone https://github.com/ggerganov/llama.cpp.git -b b4523
cd llama.cpp
mkdir -p build
cd build
source /opt/intel/oneapi/setvars.sh
cmake .. -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DLLAMA_CURL=ON -DGGML_CCACHE=OFF -DGGML_NATIVE=ON
cmake --build . --config Release -j -v
cmake --install . --prefix /tmp/llama-cpp
cd /tmp/llama-cpp
tar -cvf llama-cpp-bundle ./
```

## Minimal packages for build -

intel-opencl g++ cmake git tar libcurl-devel intel-oneapi-mkl-sycl-devel intel-oneapi-dnnl-devel intel-oneapi-compiler-dpcpp-cpp

## Build with Level Zero instead of OpenCL

g++ cmake git libcurl-devel intel-oneapi-mkl-sycl-devel intel-oneapi-dnnl-devel intel-oneapi-compiler-dpcpp-cpp intel-level-zero oneapi-level-zero oneapi-level-zero-devel intel-compute-runtime

Run LLama.CPP in workspace -

```bash
podman run -it --name llama --rm --device /dev/dri/renderD128 -p 8080:8080 --entrypoint /bin/bash -v /projects/model-dir:/model-dir:Z quay.io/cgruver0/che/my-code-assistant:latest

source /opt/intel/oneapi/setvars.sh
llama-server --model /model-dir/models/ollama/granite-code:3b --host 0.0.0.0 --n-gpu-layers 999 --flash-attn --ctx-size 32768 --jinja
```
