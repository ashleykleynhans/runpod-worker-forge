# Install Stable Diffusion WebUI Forge on your Network Volume

1. [Create a RunPod Account](https://runpod.io?ref=2xxro4sy).
2. Create a [RunPod Network Volume](https://www.runpod.io/console/user/storage).
3. Attach the Network Volume to a Secure Cloud [GPU pod](https://www.runpod.io/console/gpu-secure-cloud).
4. Select a light-weight template such as RunPod Pytorch.
5. Deploy the GPU Cloud pod.
6. Once the pod is up, open a Terminal and install the required
   dependencies. This can either be done by using the installation
   script, or manually.

## Automatic Installation Script

You can run this automatic installation script which will
automatically install all of the dependencies that get installed
manually below, and then you don't need to follow any of the
manual instructions.

```bash
wget https://raw.githubusercontent.com/ashleykleynhans/runpod-worker-forge/main/scripts/install.sh
chmod +x install.sh
./install.sh
```

## Manual Installation

You only need to complete the steps below if you did not run the
automatic installation script above.

1. Install the Stable Diffusion WebUI Forge:
```bash
# Clone the repo
cd /workspace
git clone --depth=1 https://github.com/lllyasviel/stable-diffusion-webui-forge.git

# Upgrade Python
apt update
apt -y upgrade

# Ensure Python version is 3.10.12
python3 -V

# Create and activate venv
cd stable-diffusion-webui-forge
python3 -m venv /workspace/venv
source /workspace/venv/bin/activate

# Install Torch and xformers
pip3 install --no-cache-dir torch==2.0.1+cu118 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip3 install --no-cache-dir xformers==0.0.22

# Install Stable Diffusion WebUI Forge
wget https://raw.githubusercontent.com/ashleykleynhans/runpod-worker-forge/main/install-forge.py
python3 -m install-forge --skip-torch-cuda-test

# Clone the ReActor Extension
cd /workspace/stable-diffusion-webui-forge
git clone https://github.com/Gourieff/sd-webui-reactor.git extensions/sd-webui-reactor
git checkout v0.6.1

# Clone the After Detailer Extension
git clone --depth=1 https://github.com/Bing-su/adetailer.git extensions/adetailer

# Install dependencies for ReActor
cd /workspace/stable-diffusion-webui-forge/extensions/sd-webui-reactor
pip3 install -r requirements.txt
pip3 install onnxruntime-gpu

# Install dependencies for After Detailer
cd /workspace/stable-diffusion-webui-forge/extensions/adetailer
python3 -m install

# Install the model for ReActor
mkdir -p /workspace/stable-diffusion-webui-forge/models/insightface
cd /workspace/stable-diffusion-webui-forge/models/insightface
wget https://github.com/facefusion/facefusion-assets/releases/download/models/inswapper_128.onnx

# Configure ReActor to use the GPU instead of CPU
echo "CUDA" > /workspace/stable-diffusion-webui-forge/extensions/sd-webui-reactor/last_device.txt
```
2. Install the Serverless dependencies:
```bash
cd /workspace/stable-diffusion-webui-forge
pip3 install huggingface_hub runpod
```
3. Download some models, for example `SDXL` and `Deliberate v2`:
```bash
cd /workspace/stable-diffusion-webui-forge/models/Stable-diffusion
wget https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors
wget https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors
wget -O deliberate_v2.safetensors https://huggingface.co/XpucT/Deliberate/resolve/main/Deliberate_v2.safetensors
```
4. Download VAEs for SD 1.5 and SDXL:
```bash
cd /workspace/stable-diffusion-webui-forge/models/VAE
wget https://huggingface.co/stabilityai/sd-vae-ft-mse-original/resolve/main/vae-ft-mse-840000-ema-pruned.safetensors
wget https://huggingface.co/madebyollin/sdxl-vae-fp16-fix/resolve/main/sdxl_vae.safetensors
```
5. Download ControlNet models, for example `canny` for SD 1.5 as well as SDXL:
```bash
mkdir -p /workspace/stable-diffusion-webui-forge/models/ControlNet
cd /workspace/stable-diffusion-webui-forge/models/ControlNet
wget https://huggingface.co/lllyasviel/ControlNet-v1-1/resolve/main/control_v11p_sd15_canny.pth
wget https://huggingface.co/lllyasviel/sd_control_collection/resolve/main/diffusers_xl_canny_full.safetensors
```
6. Download InstantID ControlNet models:
```bash
wget -O ip-adapter_instant_id_sdxl.bin "https://huggingface.co/InstantX/InstantID/resolve/main/ip-adapter.bin?download=true"
wget -O control_instant_id_sdxl.safetensors"https://huggingface.co/InstantX/InstantID/resolve/main/ControlNetModel/diffusion_pytorch_model.safetensors?download=true"
```
7. Create logs directory:
```bash
mkdir -p /workspace/logs
```
8. Install config files:
```bash
cd /workspace/stable-diffusion-webui-forge
rm webui-user.sh config.json ui-config.json
wget https://raw.githubusercontent.com/ashleykleynhans/runpod-worker-forge/main/webui-user.sh
wget https://raw.githubusercontent.com/ashleykleynhans/runpod-worker-forge/main/config.json
wget https://raw.githubusercontent.com/ashleykleynhans/runpod-worker-forge/main/ui-config.json
```
9. Run the Web UI:
```bash
deactivate
export HF_HOME="/workspace"
cd /workspace/stable-diffusion-webui-forge
./webui.sh -f
```
10. Wait for the Web UI to start up, and download the models. You shoud
    see something like this when it is ready:
```
Model loaded in 16.9s (calculate hash: 8.0s, load weights from disk: 0.4s, create model: 2.1s, apply weights to model: 2.6s, apply half(): 2.6s, move model to device: 0.7s, calculate empty prompt: 0.3s).
```
11. Press Ctrl-C to exit, and then you can terminate the pod.
