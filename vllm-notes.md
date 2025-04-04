# vLLM Notes

```bash
podman build -t nexus.clg.lab:5002/dev-spaces/vllm-dev:latest ./workspace-image
```

```bash
python -m pip install --upgrade pip

python -m pip install torch==2.5.1+cxx11.abi torchvision==0.20.1+cxx11.abi torchaudio==2.5.1+cxx11.abi intel-extension-for-pytorch==2.5.10+xpu oneccl_bind_pt==2.5.0+xpu --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/

python -m pip install triton-xpu==3.2.0b1

cd /projects/vllm

cat << EOF > requirements-xpu.txt
-r requirements-common.txt

ray>=2.9
cmake>=3.26
ninja
packaging
setuptools-scm>=8
setuptools>=75.8.0
wheel
jinja2
EOF

find ~/.local -name mha_fusion.py

sed -i '326d' ${HOME}/.local/lib/python3.12/site-packages/intel_extension_for_pytorch/transformers/models/reference/fusions/mha_fusion.py

VLLM_TARGET_DEVICE=xpu python -m pip install .
```

```bash
export VLLM_LOGGING_LEVEL=DEBUG

vllm serve --host 0.0.0.0 --port 8080 --device xpu --gpu-memory-utilization 0.3 Qwen/Qwen2.5-1.5B-Instruct
```

```bash
curl -k https://cgruver-vllm-intel-gpu-vllm.apps.region-01.clg.lab/v1/completions -X POST -H "Content-Type: application/json" -d '{"model":"Qwen/Qwen2.5-1.5B-Instruct","prompt": "San Francisco is a","max_tokens": 7,"temperature": 0}'

curl -k https://cgruver-vllm-intel-gpu-vllm.apps.region-01.clg.lab/v1/chat/completions -X POST -H "Content-Type: application/json" -d '{"model": "Qwen/Qwen2.5-1.5B-Instruct","messages": [{"role": "system", "content": "You are a helpful assistant."},{"role": "user", "content": "Who won the world series in 2020?"}]}'

curl -k https://cgruver-vllm-intel-gpu-vllm.apps.region-01.clg.lab/v1/models

```


```
python -m pip install pybind11

git clone https://github.com/intel/intel-xpu-backend-for-triton.git
cd intel-xpu-backend-for-triton

scripts/install-pytorch.sh --source

scripts/compile-triton.sh
```
