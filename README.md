<div align="center">
<img src="https://github.com/alpa-projects/alpa/blob/main/docs/logo/alpa-logo-cropped.png" alt="logo" width="250"></img>
<br></br>
</div>

[![CI](https://github.com/alpa-projects/alpa/actions/workflows/ci.yml/badge.svg)](https://github.com/alpa-projects/alpa/actions/workflows/ci.yml)
[![Build Jaxlib](https://github.com/alpa-projects/alpa/actions/workflows/build_jaxlib.yml/badge.svg)](https://github.com/alpa-projects/alpa/actions/workflows/build_jaxlib.yml)

[**Documentation**](https://alpa-projects.github.io) | [**Slack**](https://forms.gle/YEZTCrtZD6EAVNBQ7)

Alpa is a system for training and serving large-scale neural networks.

Scaling neural networks to hundreds of billions of parameters has enabled dramatic breakthroughs such as GPT-3, but training and serving these large-scale neural networks require complicated distributed system techniques.
Alpa aims to automate large-scale distributed training and serving with just a few lines of code.

The key features of Alpa include:  

💻 **Automatic Parallelization**. Alpa automatically parallelizes users' single-device code on distributed clusters with data, operator, and pipeline parallelism. 

🚀 **Excellent Performance**. Alpa achieves linear scaling on training models with billions of parameters on distributed clusters.

✨ **Tight Integration with Machine Learning Ecosystem**. Alpa is backed by open-source, high-performance, and production-ready libraries such as [Jax](https://github.com/google/jax), [XLA](https://www.tensorflow.org/xla), and [Ray](https://github.com/ray-project/ray).

## Serving
Alpa provides a free, unlimited OPT-175B text generation service.
Try the service at [https://opt.alpa.ai/](https://opt.alpa.ai/) and share your [prompting results](examples/llm_serving/service/img.png)!

The code below shows how to use huggingface/transformers interface and Alpa distributed backend for large model inference.
Detailed documentation is in [Serving OPT-175B using Alpa](https://alpa.ai/tutorials/opt_serving.html).

```python
from transformers import AutoTokenizer
from llm_serving.model.wrapper import get_model

# Load the tokenizer
tokenizer = AutoTokenizer.from_pretrained("facebook/opt-2.7b")
tokenizer.add_bos_token = False

# Load the model. Alpa automatically downloads the weights to the specificed path
model = get_model(model_name="alpa/opt-2.7b", path="~/opt_weights/")

# Generate
prompt = "Paris is the capital city of"

input_ids = tokenizer(prompt, return_tensors="pt").input_ids
output = model.generate(input_ids=input_ids, max_length=256, do_sample=True)
generated_string = tokenizer.batch_decode(output, skip_special_tokens=True)

print(generated_string)
```

## Training
Use Alpa's decorator ``@parallelize`` to scale your single-device training code to distributed clusters.
Check out the [documentation](https://alpa-projects.github.io) site and
[examples](https://github.com/alpa-projects/alpa/tree/main/examples) folder
for installation instructions, tutorials, examples, and more.

```python
import alpa

# Parallelize the training step in Jax by simply using a decorator
@alpa.parallelize
def train_step(model_state, batch):
    def loss_func(params):
        out = model_state.forward(params, batch["x"])
        return jnp.mean((out - batch["y"]) ** 2)

    grads = grad(loss_func)(model_state.params)
    new_model_state = model_state.apply_gradient(grads)
    return new_model_state

# The training loop now automatically runs on your designated cluster
model_state = create_train_state()
for batch in data_loader:
    model_state = train_step(model_state, batch)
```

## Learning more
- [OSDI 2022 paper](https://arxiv.org/abs/2201.12023)
- [Google AI blog](https://ai.googleblog.com/2022/05/alpa-automated-model-parallel-deep.html)
- [Talk slides](https://docs.google.com/presentation/d/1CQ4S1ff8yURk9XmL5lpQOoMMlsjw4m0zPS6zYDcyp7Y/edit?usp=sharing)
- [ICML 2022 Big Model Tutorial slides](https://sites.google.com/view/icml-2022-big-model/home)
- [ICML 2022 Big Model Tutorial video recording](https://icml.cc/virtual/2022/tutorial/18440)
- [Prof. Ion Stoica introduces the system](https://www.youtube.com/watch?v=qzYoMldlyoA)

## Getting Involved
- Connect to Alpa developers via the [Alpa slack](https://forms.gle/YEZTCrtZD6EAVNBQ7).
- Please read the [contributor guide](https://alpa-projects.github.io/developer/developer_guide.html) if you are interested in contributing code.

## License
Alpa is licensed under the [Apache-2.0 license](https://github.com/alpa-projects/alpa/blob/main/LICENSE).



# Helper on how to install for LLM serving

Install instructions: https://alpa.ai/install.html

## Pre install activities

- Install CUDA and cuDNN first:

CUDA:
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.0.0/local_installers/cuda-repo-ubuntu2204-12-0-local_12.0.0-525.60.13-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-12-0-local_12.0.0-525.60.13-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-12-0-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda

cuDNN:
https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html

NVIDIA-CUDA-toolkit:
sudo apt install nvidia-cuda-toolkit

NCCL library:
https://docs.nvidia.com/deeplearning/nccl/install-guide/index.html#:~:text=Installing%20NCCL%20on%20Ubuntu%20requires,repository%20and%20a%20network%20repository.


jaxlib:

From wheel: https://alpa.ai/wheels.html
download manually, then pip install XXX

Extra installs for serving models:
pip install "transformers<=4.23.1" fastapi uvicorn omegaconf jinja2
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu116

Install llm_serving package from alpa:
cd alpa/examples
pip install -e .

Must have Python 3.9 or 3.8


## Install


virtualenv -p python3.9 env
source ./env/bin/activate
pip install cupy-cuda116
--> install pre dependencies (cuda, cudnn, nvidia toolkit, nccl)
pip install alpa
--> install jaxlib
pip install "transformers<=4.23.1" fastapi uvicorn omegaconf jinja2
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu116
cd examples
pip install -e .

Test installation:

ray start --head
python -m alpa.test_install


## ISSUES

cupy_backends.cuda.libs.nccl.NcclError: NCCL_ERROR_UNHANDLED_CUDA_ERROR: unhandled cuda error, it is mainly due to the compatibility issues between CUDA, NCCL, and GPU driver versions.
    https://github.com/alpa-projects/alpa/issues/496

SOLUTION:
python -m cupyx.tools.install_library --cuda 11.6 --library nccl


Problem: FileNotFoundError: [Errno 2] No such file or directory: '/home/bundenth/opt_weights/opt-125m-np/decoder.embed_tokens.weight'

SOLUTION:
All nodes must have access to the same file path (including username!)


Problem BLAS:

(MeshHostWorker pid=7385, ip=192.168.0.27) 2023-01-16 18:15:42.416882: E external/org_tensorflow/tensorflow/compiler/xla/stream_executor/cuda/cuda_blas.cc:219] failed to create cublas handle: cublas error
(MeshHostWorker pid=7385, ip=192.168.0.27) 2023-01-16 18:15:42.416914: E external/org_tensorflow/tensorflow/compiler/xla/stream_executor/cuda/cuda_blas.cc:221] Failure to initialize cublas may be due to OOM (cublas needs some free memory when you initialize it, and your deep-learning framework may have preallocated more than its fair share), or may be because this binary was not built with support for the GPU in your machine.
(MeshHostWorker pid=7385, ip=192.168.0.27) 2023-01-16 18:15:42.416966: E external/org_tensorflow/tensorflow/compiler/xla/pjrt/pjrt_stream_executor_client.cc:2156] Execution of replica 0 failed: INTERNAL: Attempting to perform BLAS operation using StreamExecutor without BLAS support

SOLUTION (pre allocate less mem?)

export XLA_PYTHON_CLIENT_MEM_FRACTION=0.6