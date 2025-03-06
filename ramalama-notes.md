## Ramalama

pip install ramalama
export PATH=${PATH}:${HOME}/.local/bin

ramalama --gpu --image quay.io/cgruver0/llama-cpp-intel-gpu:latest run granite-code:3b

## Build and Install locally
```bash
pip install . --no-deps --root / --prefix ${HOME}/ramalama
```


Run in workspace -

```bash
export PYTHONPATH=${HOME}/ramalama/lib/python3.12/site-packages
export PATH=${PATH}:${HOME}/ramalama/bin
export RAMLAMA_STORE=/projects/model-dir

ramalama serve --ngl 33 --ctx-size 32768 --device /dev/dri/renderD128 --network podman granite-code:3b
```
