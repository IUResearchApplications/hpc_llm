# hpc_llm
## Delivering Large Language Model Platforms with HPC



## Installation & Setup

```
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
```

### cuBLAS Install
```
module load python/gpu/3.10.10
make LLAMA_CUBLAS=1
https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#cublas
```

### OpenBLAS Install
```
module load openblas
export PKG_CONFIG_PATH=/N/soft/rhel8/openblas/gnu/0.3.13/lib/pkgconfig
make LLAMA_OPENBLAS=1
https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#blas-builds
```



### Converting Llama2 Model to .gguf and Quantization
```
module load hpc_llm/.1.0.lua
module load python/gpu/3.10.10

python -m pip install -r requirements.txt
python convert.py models/llama-2-13b/ --ctx 4096
```

You need to make target directories before running this:
```
quantize llama-2-7b/ggml-model-f16.gguf ../quantized/llama-2-7b/ggml-model-Q4_K_M.gguf Q4_K_M

```

Available quantization methods available at https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#quantization


## Usage
Llama.cpp provides the command `main`, which is used to run your selected model in both interactive and automated manners. To simplify common tasks such as asking questions and summarizing documents, our `tellme` and `summarize` bash script utilities are also provided. These are tailored to the CPU-only or GPU-accelerated utility the command is issued from.

### tellme
Answers general questions. Type your question or request after the command `tellme`
```
tellme Why is the sky blue?
```
### summarize
Provides simple summaries for raw text file(s). Some test files and templates are available at `/N/project/HPC_LLM/HPC_LLM/examples`
```
summarize -f filename.txt # summarize will enter a single text file into a template for you
summarize -p filename.txt # use this if your text file is already formatted into a query template
summarize -d /directory/of/files # summarize will enter all files in a directory into a single template for you
```

You can also follow the below instructions to use `main` directly



## Customizing Your Usage
There are many options you can use to adjust your run when using `main` directly. A specific model can be specified with `-m modelpath`.  We provide llama2 and llama2-chat in 7B, 13B, and 70B sizes, both quantized and unquantized, at `/N/project/HPC_LLM/HPC_LLM/models`

Unquantized models are the full model, converted for use with llama.cpp. Quantized models are converted models that perform some or all of the operations with reduced precision rather than full precision (floating point) values, and occupy less system or GPU memory.


### Useful flags
`-n N` requests a response from Llama containing N tokens. `-n -2` requests that the LLM continue responding until you stop it with CTRL+C, or until its context window is filled.

`-t N` is the number of threads to use during generation (default: 22). To limit usage on RED and login nodes, -t 4 is used in the utility commands.

`-ngl N` diverts N layers to the GPUs
|Model Size | Max. Layers|
| --------  | ---------- |
|7B         | 33         |
|30B        | 41         |
|70B        | 81         |

`--color` colors the output to distinguish between prompt, user input, and generated text.
`-i, --interactive` runs in interactive mode. If TEXT is provided after flag, TEXT will be ingested immediately. Control+C at any time interrupts the LLM and lets user respond.
`--interactive-first` runs in interactive mode and waits for input right away, the user is the one to launch the question even if TEXT is provided after flag (it is auto-populated in window, however). Control+C at any time interrupts the LLM and lets user respond.

For more, see `main --help`or visit [llama.cpp](https://github.com/ggerganov/llama.cpp).


## Benchmarking
### A100 40GB Usage - Big Red 200
```
module load hpc_llm/gpu/.1.0
export OMP_NUM_THREADS=40
main -m /N/project/HPC_LLM/HPC_LLM/models/quantized/llama-2-<7b/13b/70b>/ggml-model-Q4_K_M.gguf -n -2 -t 40 -ngl <33/41/81> --log-new -f /N/project/HPC_LLM/HPC_LLM/examples/metrics_prompt.txt
```

### V100 32GB Usage - Quartz
```
module load hpc_llm/gpu/.1.0
export OMP_NUM_THREADS=40
main -m /N/project/HPC_LLM/HPC_LLM/models/quantized/llama-2-<7b/13b/70b>/ggml-model-Q4_K_M.gguf -n -2 -t 40 -ngl <33/41/65> --log-new -f /N/project/HPC_LLM/HPC_LLM/examples/metrics_prompt.txt
```
### No GPUs - Research Desktop
Intended workflow usage:
```
module load hpc_llm/.1.0
OMP_NUM_THREADS=5
main -m /N/project/HPC_LLM/HPC_LLM/models/quantized/llama-2-<7b/13b/70b>/ggml-model-Q4_K_M.gguf -n -2 -t 5 --log-new -f /N/project/HPC_LLM/HPC_LLM/examples/metrics_prompt.txt
```
Full usage:
```
module load hpc_llm/.1.0
export OMP_NUM_THREADS=40
main -m /N/project/HPC_LLM/HPC_LLM/models/quantized/llama-2-<7b/13b/70b>/ggml-model-Q4_K_M.gguf -n -2 -t 40 --log-new -f /N/project/HPC_LLM/HPC_LLM/examples/metrics_prompt.txt
```

### GH200 480GB - Testbench
```
export OMP_NUM_THREADS=40
main -m /N/project/HPC_LLM/HPC_LLM/models/quantized/llama-2-<7b/13b/70b>/ggml-model-Q4_K_M.gguf -n -2 -t 40 -ngl <33/41/81> --log-new -f /N/project/HPC_LLM/HPC_LLM/examples/metrics_prompt.txt
```
```
main -m /data/HPC_LLM/models/quantized/llama-2-7b/ggml-model-Q4_K_M.gguf -n -2 -t 40 -ngl 33  -f /data/HPC_LLM/examples/metrics_prompt.txt
```
