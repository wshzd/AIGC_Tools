# llama.cpp_zh

Georgi Gerganov发布了一个名为「llama.cpp」的项目，号称没有GPU也能跑LLaMA，关于这个项目的介绍，参考计算机科学家Simon Willison的博文[Large language models are having their Stable Diffusion moment](https://simonwillison.net/2023/Mar/11/llama/) ，项目链接[github](https://github.com/ggerganov/llama.cpp)

**Note**：目前只支持mac和linux，下面是这两个平台的运行步骤

## Mac

**第一步：下载模型**

首先要做的就是下载LLaMA模型。

你可以通过官方的表格向Meta提交申请，或者从网友分享的链接里直接获取。

总之，完成后你会看到下面这堆东西：

![图片](https://mmbiz.qpic.cn/mmbiz_png/UicQ7HgWiaUb3icODOSBurJKf4B5fhNsNIC9dJO15lPbZHONnCibWJEsI4a4vjRd900U3Htgq0zsenHJ3dTzBV2WKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

正如你所看到的，不同的模型都在不同的文件夹里。每个模型都有一个params.json，包含关于该模型的细节。比如：

![图片](https://mmbiz.qpic.cn/mmbiz_png/UicQ7HgWiaUb3icODOSBurJKf4B5fhNsNICQgpxOvPtLSJazibryN4eGfhQPagxG3ugyHZkAeEtAichxg1vgib3sHK7A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**第二步：安装依赖项**

首先，你需要安装Xcode来编译C++项目。

```
xcode-select --install
```

接下来，是构建C++项目的依赖项（pkgconfig和cmake）。

```
brew install pkgconfig cmake
```

在环境的配置上，假如你用的是Python 3.11，则可以创建一个虚拟环境：

```
/opt/homebrew/bin/python3.11 -m venv venv
```

然后激活venv。（如果是fish以外的shell，只要去掉.fish后缀即可）

```
. venv/bin/activate.fish
```

最后，安装Torch。

```
pip3 install --pre torch torchvision --extra-index-url https://download.pytorch.org/whl/nightly/cpu
```

如果你对利用新的Metal性能着色器（MPS）后端进行GPU训练加速感兴趣，可以通过运行以下程序来进行验证。但这不是在M1上运行LLaMA的必要条件。

```
pythonPython 3.11.2 (main, Feb 16 2023, 02:55:59) [Clang 14.0.0 (clang-1400.0.29.202)] on darwinType "help", "copyright", "credits" or "license" for more information.>>> import torch; torch.backends.mps.is_available()True
```

**第三步：编译LLaMA CPP**

```
git clone git@github.com:ggerganov/llama.cpp.git
```

在安装完所有的依赖项后，你可以运行make：

```
makeI llama.cpp build info:I UNAME_S:  DarwinI UNAME_P:  armI UNAME_M:  arm64I CFLAGS:   -I.              -O3 -DNDEBUG -std=c11   -fPIC -pthread -DGGML_USE_ACCELERATEI CXXFLAGS: -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthreadI LDFLAGS:   -framework AccelerateI CC:       Apple clang version 14.0.0 (clang-1400.0.29.202)I CXX:      Apple clang version 14.0.0 (clang-1400.0.29.202)cc  -I.              -O3 -DNDEBUG -std=c11   -fPIC -pthread -DGGML_USE_ACCELERATE   -c ggml.c -o ggml.oc++ -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread -c utils.cpp -o utils.oc++ -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread main.cpp ggml.o utils.o -o main  -framework Accelerate./main -husage: ./main [options]options:  -h, --help            show this help message and exit  -s SEED, --seed SEED  RNG seed (default: -1)    -t N, --threads N     number of threads to use during computation (default: 4)    -p PROMPT, --prompt PROMPT                        prompt to start generation with (default: random)    -n N, --n_predict N   number of tokens to predict (default: 128)    --top_k N             top-k sampling (default: 40)    --top_p N             top-p sampling (default: 0.9)    --temp N              temperature (default: 0.8)    -b N, --batch_size N  batch size for prompt processing (default: 8)    -m FNAME, --model FNAME                        model path (default: models/llama-7B/ggml-model.bin)c++ -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread quantize.cpp ggml.o utils.o -o quantize  -framework Accelerate
```

**第四步：转换模型**

假设你已经把模型放在llama.cpp repo中的models/下。

```
python convert-pth-to-ggml.py models/7B 1
```

那么，应该会看到像这样的输出：

```
{'dim': 4096, 'multiple_of': 256, 'n_heads': 32, 'n_layers': 32, 'norm_eps': 1e-06, 'vocab_size': 32000}n_parts =  1Processing part  0Processing variable: tok_embeddings.weight with shape:  torch.Size([32000, 4096])  and type:  torch.float16Processing variable: norm.weight with shape:  torch.Size([4096])  and type:  torch.float16  Converting to float32Processing variable: output.weight with shape:  torch.Size([32000, 4096])  and type:  torch.float16Processing variable: layers.0.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16Processing variable: layers.0.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16Processing variable: layers.0.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16Processing variable: layers.0.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16Processing variable: layers.0.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16Processing variable: layers.0.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16Processing variable: layers.0.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16Processing variable: layers.0.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16...Done. Output file: models/7B/ggml-model-f16.bin, (part  0 )
```

下一步将是进行量化处理：

```
./quantize ./models/7B/ggml-model-f16.bin ./models/7B/ggml-model-q4_0.bin 2
```

输出如下：

```
llama_model_quantize: loading model from './models/7B/ggml-model-f16.bin'llama_model_quantize: n_vocab = 32000llama_model_quantize: n_ctx   = 512llama_model_quantize: n_embd  = 4096llama_model_quantize: n_mult  = 256llama_model_quantize: n_head  = 32llama_model_quantize: n_layer = 32llama_model_quantize: f16     = 1...layers.31.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MBlayers.31.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MBllama_model_quantize: model size  = 25705.02 MBllama_model_quantize: quant size  =  4017.27 MBllama_model_quantize: hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
main: quantize time = 29389.45 msmain:    total time = 29389.45 ms
```

**第五步：运行模型**

```
./main -m ./models/7B/ggml-model-q4_0.bin \        -t 8 \        -n 128 \        -p 'The first president of the USA was '
```

```
main: seed = 1678615879llama_model_load: loading model from './models/7B/ggml-model-q4_0.bin' - please wait ...llama_model_load: n_vocab = 32000llama_model_load: n_ctx   = 512llama_model_load: n_embd  = 4096llama_model_load: n_mult  = 256llama_model_load: n_head  = 32llama_model_load: n_layer = 32llama_model_load: n_rot   = 128llama_model_load: f16     = 2llama_model_load: n_ff    = 11008llama_model_load: n_parts = 1llama_model_load: ggml ctx size = 4529.34 MBllama_model_load: memory_size =   512.00 MB, n_mem = 16384llama_model_load: loading model part 1/1 from './models/7B/ggml-model-q4_0.bin'llama_model_load: .................................... donellama_model_load: model size =  4017.27 MB / num tensors = 291main: prompt: 'The first president of the USA was 'main: number of tokens in prompt = 9     1 -> ''  1576 -> 'The'   937 -> ' first'  6673 -> ' president'   310 -> ' of'   278 -> ' the'  8278 -> ' USA'   471 -> ' was' 29871 -> ' 'sampling parameters: temp = 0.800000, top_k = 40, top_p = 0.950000
The first president of the USA was 57 years old when he assumed office (George Washington). Nowadays, the US electorate expects the new president to be more young at heart. President Donald Trump was 70 years old when he was inaugurated. In contrast to his predecessors, he is physically fit, healthy and active. And his fitness has been a prominent theme of his presidency. During the presidential campaign, he famously said he would be the “most active president ever” — a statement Trump has not yet achieved, but one that fits his approach to the office. His tweets demonstrate his physical activity.
main: mem per token = 14434244 bytesmain:     load time =  1311.74 msmain:   sample time =   278.96 msmain:  predict time =  7375.89 ms / 54.23 ms per tokenmain:    total time =  9216.61 ms
```

## linux

**第一步：下载模型**

可以参考Mac **第一步：下载模型**

**第二步：安装依赖项**

一般我会使用conda创建一个虚拟机，在虚拟机里面再创建llama.cpp所需要的依赖项

创建虚拟机的命令如下（optional）：

```
create -n 虚拟机名称 python==python版本（比如3.10）
```

```
# install Python dependencies
python3 -m pip install torch numpy sentencepiece
```

**第三步：编译LLaMA CPP**

```
# build this repo
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make
```

```
# 输出编译信息如下：
I llama.cpp build info:
I UNAME_S:  Linux
I UNAME_P:  x86_64
I UNAME_M:  x86_64
I CFLAGS:   -I.              -O3 -DNDEBUG -std=c11   -fPIC -pthread -mavx -mavx2 -mfma -mf16c -msse3
I CXXFLAGS: -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread
I LDFLAGS:
I CC:       gcc (GCC) 5.4.0
I CXX:      g++ (GCC) 5.4.0

/usr/local/gcc-5.4.0/bin/gcc  -I.              -O3 -DNDEBUG -std=c11   -fPIC -pthread -mavx -mavx2 -mfma -mf16c -msse3   -c ggml.c -o ggml.o
/usr/local/gcc-5.4.0/bin/g++ -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread main.cpp ggml.o utils.o -o main
./main -h
usage: ./main [options]

options:
  -h, --help            show this help message and exit
  -i, --interactive     run in interactive mode
  --interactive-start   run in interactive mode and poll user input at startup
  -r PROMPT, --reverse-prompt PROMPT
                        in interactive mode, poll user input upon seeing PROMPT
  --color               colorise output to distinguish prompt and user input from generations
  -s SEED, --seed SEED  RNG seed (default: -1)
  -t N, --threads N     number of threads to use during computation (default: 4)
  -p PROMPT, --prompt PROMPT
                        prompt to start generation with (default: random)
  -f FNAME, --file FNAME
                        prompt file to start generation.
  -n N, --n_predict N   number of tokens to predict (default: 128)
  --top_k N             top-k sampling (default: 40)
  --top_p N             top-p sampling (default: 0.9)
  --repeat_last_n N     last n tokens to consider for penalize (default: 64)
  --repeat_penalty N    penalize repeat sequence of tokens (default: 1.3)
  --temp N              temperature (default: 0.8)
  -b N, --batch_size N  batch size for prompt processing (default: 8)
  -m FNAME, --model FNAME
                        model path (default: models/llama-7B/ggml-model.bin)

/usr/local/gcc-5.4.0/bin/g++ -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread quantize.cpp ggml.o utils.o -o quantize
```

```
# obtain the original LLaMA model weights and place them in ./models
ls ./models
65B 30B 13B 7B tokenizer_checklist.chk tokenizer.model
```

**第四步：转换模型**

```
# convert the 7B model to ggml FP16 format
python3 convert-pth-to-ggml.py models/7B/ 1
```

```
{'dim': 4096, 'multiple_of': 256, 'n_heads': 32, 'n_layers': 32, 'norm_eps': 1e-06, 'vocab_size': 32000}
n_parts =  1
Processing part  0
Processing variable: tok_embeddings.weight with shape:  torch.Size([32000, 4096])  and type:  torch.float16
Processing variable: norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: output.weight with shape:  torch.Size([32000, 4096])  and type:  torch.float16
Processing variable: layers.0.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.0.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.0.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.0.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.0.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.0.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.0.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.0.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.0.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.1.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.1.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.1.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.1.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.1.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.1.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.1.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.1.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.1.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.2.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.2.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.2.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.2.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.2.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.2.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.2.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.2.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.2.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.3.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.3.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.3.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.3.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.3.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.3.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.3.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.3.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.3.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.4.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.4.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.4.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.4.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.4.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.4.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.4.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.4.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.4.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.5.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.5.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.5.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.5.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.5.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.5.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.5.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.5.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.5.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.6.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.6.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.6.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.6.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.6.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.6.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.6.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.6.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.6.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.7.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.7.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.7.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.7.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.7.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.7.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.7.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.7.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.7.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.8.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.8.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.8.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.8.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.8.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.8.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.8.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.8.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.8.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.9.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.9.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.9.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.9.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.9.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.9.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.9.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.9.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.9.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.10.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.10.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.10.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.10.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.10.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.10.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.10.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.10.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.10.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.11.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.11.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.11.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.11.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.11.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.11.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.11.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.11.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.11.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.12.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.12.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.12.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.12.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.12.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.12.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.12.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.12.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.12.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.13.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.13.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.13.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.13.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.13.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.13.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.13.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.13.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.13.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.14.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.14.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.14.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.14.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.14.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.14.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.14.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.14.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.14.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.15.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.15.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.15.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.15.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.15.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.15.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.15.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.15.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.15.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.16.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.16.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.16.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.16.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.16.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.16.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.16.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.16.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.16.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.17.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.17.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.17.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.17.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.17.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.17.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.17.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.17.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.17.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.18.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.18.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.18.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.18.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.18.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.18.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.18.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.18.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.18.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.19.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.19.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.19.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.19.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.19.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.19.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.19.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.19.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.19.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.20.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.20.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.20.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.20.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.20.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.20.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.20.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.20.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.20.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.21.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.21.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.21.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.21.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.21.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.21.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.21.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.21.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.21.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.22.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.22.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.22.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.22.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.22.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.22.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.22.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.22.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.22.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.23.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.23.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.23.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.23.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.23.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.23.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.23.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.23.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.23.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.24.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.24.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.24.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.24.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.24.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.24.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.24.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.24.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.24.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.25.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.25.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.25.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.25.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.25.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.25.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.25.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.25.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.25.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.26.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.26.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.26.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.26.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.26.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.26.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.26.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.26.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.26.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.27.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.27.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.27.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.27.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.27.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.27.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.27.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.27.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.27.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.28.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.28.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.28.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.28.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.28.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.28.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.28.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.28.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.28.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.29.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.29.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.29.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.29.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.29.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.29.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.29.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.29.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.29.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.30.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.30.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.30.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.30.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.30.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.30.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.30.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.30.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.30.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.31.attention.wq.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.31.attention.wk.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.31.attention.wv.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.31.attention.wo.weight with shape:  torch.Size([4096, 4096])  and type:  torch.float16
Processing variable: layers.31.feed_forward.w1.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.31.feed_forward.w2.weight with shape:  torch.Size([4096, 11008])  and type:  torch.float16
Processing variable: layers.31.feed_forward.w3.weight with shape:  torch.Size([11008, 4096])  and type:  torch.float16
Processing variable: layers.31.attention_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Processing variable: layers.31.ffn_norm.weight with shape:  torch.Size([4096])  and type:  torch.float16
  Converting to float32
Done. Output file: models/7B//ggml-model-f16.bin, (part  0 )
```

```
# quantize the model to 4-bits
./quantize ./models/7B/ggml-model-f16.bin ./models/7B/ggml-model-q4_0.bin 2
```

```
llama_model_quantize: loading model from './models/7B/ggml-model-f16.bin'
llama_model_quantize: n_vocab = 32000
llama_model_quantize: n_ctx   = 512
llama_model_quantize: n_embd  = 4096
llama_model_quantize: n_mult  = 256
llama_model_quantize: n_head  = 32
llama_model_quantize: n_layer = 32
llama_model_quantize: f16     = 1
                           tok_embeddings.weight - [ 4096, 32000], type =    f16 quantizing .. size =   500.00 MB ->    78.12 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                                     norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                                   output.weight - [ 4096, 32000], type =    f16 quantizing .. size =   500.00 MB ->    78.12 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.136 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                    layers.0.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.020 0.015 0.025 0.042 0.068 0.102 0.142 0.172 0.142 0.102 0.068 0.042 0.025 0.015 0.020
                    layers.0.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.021 0.015 0.027 0.045 0.070 0.103 0.139 0.159 0.139 0.104 0.070 0.045 0.027 0.015 0.021
                    layers.0.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.018 0.032 0.051 0.076 0.103 0.128 0.142 0.128 0.103 0.076 0.051 0.032 0.019 0.022
                    layers.0.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.021 0.016 0.028 0.046 0.072 0.105 0.137 0.152 0.137 0.105 0.072 0.046 0.028 0.016 0.021
                 layers.0.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.0.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                 layers.0.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                  layers.0.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.0.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.1.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.018 0.032 0.051 0.076 0.104 0.127 0.137 0.127 0.104 0.077 0.051 0.032 0.019 0.021
                    layers.1.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.021 0.018 0.031 0.051 0.076 0.104 0.128 0.138 0.128 0.104 0.076 0.051 0.032 0.018 0.021
                    layers.1.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.021 0.018 0.031 0.050 0.076 0.104 0.129 0.140 0.129 0.104 0.076 0.051 0.031 0.018 0.021
                    layers.1.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.021 0.016 0.028 0.045 0.071 0.104 0.138 0.155 0.138 0.104 0.071 0.045 0.028 0.016 0.021
                 layers.1.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.1.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.1.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.1.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.1.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.2.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.2.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.051 0.076 0.104 0.127 0.138 0.127 0.104 0.077 0.051 0.032 0.019 0.022
                    layers.2.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.136 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.2.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                 layers.2.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.2.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.2.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.2.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.2.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.3.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                    layers.3.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.136 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                    layers.3.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.3.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.3.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.3.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.3.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.3.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.3.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.4.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.4.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                    layers.4.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.135 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.4.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.126 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                 layers.4.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.4.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.4.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.4.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.4.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.5.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                    layers.5.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.5.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.5.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.5.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.5.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.5.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.5.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.5.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.6.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                    layers.6.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.6.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.6.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.6.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.6.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.6.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.6.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.6.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.7.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                    layers.7.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.7.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.7.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.7.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.7.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.7.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.7.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.7.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.8.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                    layers.8.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                    layers.8.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.8.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.8.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.8.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.078 0.052 0.033 0.019 0.022
                 layers.8.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.8.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.8.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                    layers.9.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                    layers.9.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                    layers.9.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                    layers.9.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.9.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.9.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                 layers.9.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                  layers.9.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                        layers.9.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.10.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.10.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.10.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.10.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.10.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.10.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                layers.10.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.10.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.10.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.11.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.11.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.11.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.11.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.124 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.11.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.11.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                layers.11.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.11.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.11.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.12.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.12.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.12.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.12.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.12.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.12.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                layers.12.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.12.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.12.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.13.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.13.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.13.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.13.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.124 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.13.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.13.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                layers.13.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.13.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.13.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.14.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.14.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.14.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.14.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.14.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.14.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                layers.14.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.14.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.14.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.15.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.15.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.15.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.15.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.15.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.15.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                layers.15.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.15.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.15.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.16.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.16.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.16.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.135 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.16.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.16.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.16.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.078 0.052 0.033 0.019 0.022
                layers.16.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.16.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.16.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.17.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.17.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.17.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.17.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.17.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.17.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.17.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.17.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.17.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.18.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.18.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.18.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.18.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.124 0.133 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.18.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.18.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.18.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.18.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.18.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.19.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.19.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.19.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.19.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.124 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.19.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.19.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.19.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.19.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.19.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.20.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.20.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.20.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.20.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.20.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.20.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.20.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.20.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.20.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.21.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.21.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.21.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.21.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.124 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.21.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.21.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.21.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.21.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.21.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.22.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.22.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.22.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.22.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.22.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.22.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.22.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.22.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.22.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.23.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.23.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.052 0.033 0.019 0.022
                   layers.23.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.23.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.23.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.23.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.23.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.23.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.23.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.24.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.24.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.24.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.24.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.24.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.24.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.24.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.24.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.24.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.25.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.25.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.25.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.25.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.25.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.25.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.25.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.25.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.25.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.26.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.26.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.26.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.26.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.26.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.26.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.26.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.26.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.26.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.27.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.27.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.27.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.27.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.27.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.27.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.27.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.27.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.27.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.28.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.28.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.28.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.28.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.28.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.132 0.124 0.104 0.078 0.053 0.033 0.019 0.022
                layers.28.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.28.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.28.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.28.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.29.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.29.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                   layers.29.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.29.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.29.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.29.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                layers.29.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.29.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.29.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.30.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.126 0.134 0.125 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.30.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.30.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.077 0.104 0.125 0.134 0.125 0.104 0.077 0.053 0.033 0.019 0.022
                   layers.30.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.30.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.30.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.018 0.032 0.051 0.076 0.104 0.128 0.137 0.128 0.104 0.076 0.051 0.032 0.018 0.022
                layers.30.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.30.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.30.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                   layers.31.attention.wq.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.032 0.019 0.022
                   layers.31.attention.wk.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.126 0.134 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.31.attention.wv.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.032 0.052 0.077 0.104 0.126 0.135 0.126 0.104 0.077 0.052 0.033 0.019 0.022
                   layers.31.attention.wo.weight - [ 4096,  4096], type =    f16 quantizing .. size =    64.00 MB ->    10.00 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.31.feed_forward.w1.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.133 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                layers.31.feed_forward.w2.weight - [11008,  4096], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.021 0.018 0.031 0.050 0.075 0.104 0.130 0.140 0.130 0.104 0.075 0.050 0.031 0.018 0.021
                layers.31.feed_forward.w3.weight - [ 4096, 11008], type =    f16 quantizing .. size =   172.00 MB ->    26.88 MB | hist: 0.000 0.022 0.019 0.033 0.052 0.077 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022
                 layers.31.attention_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
                       layers.31.ffn_norm.weight - [ 4096,     1], type =    f32 size =    0.016 MB
llama_model_quantize: model size  = 25705.02 MB
llama_model_quantize: quant size  =  4017.27 MB
llama_model_quantize: hist: 0.000 0.022 0.019 0.033 0.053 0.078 0.104 0.125 0.134 0.125 0.104 0.078 0.053 0.033 0.019 0.022

main: quantize time = 96426.29 ms
main:    total time = 96426.29 ms
```

**第五步：运行模型**

```
# run the inference
./main -m ./models/7B/ggml-model-q4_0.bin -t 8 -n 128
```

```
main: seed = 1678767218
llama_model_load: loading model from './models/7B/ggml-model-q4_0.bin' - please wait ...
llama_model_load: n_vocab = 32000
llama_model_load: n_ctx   = 512
llama_model_load: n_embd  = 4096
llama_model_load: n_mult  = 256
llama_model_load: n_head  = 32
llama_model_load: n_layer = 32
llama_model_load: n_rot   = 128
llama_model_load: f16     = 2
llama_model_load: n_ff    = 11008
llama_model_load: n_parts = 1
llama_model_load: ggml ctx size = 4529.34 MB
llama_model_load: memory_size =   512.00 MB, n_mem = 16384
llama_model_load: loading model part 1/1 from './models/7B/ggml-model-q4_0.bin'
llama_model_load: .................................... done
llama_model_load: model size =  4017.27 MB / num tensors = 291

main: prompt: 'They'
main: number of tokens in prompt = 2
     1 -> ''
 15597 -> 'They'

sampling parameters: temp = 0.800000, top_k = 40, top_p = 0.950000, repeat_last_n = 64, repeat_penalty = 1.300000


They are designed for the most demanding environments, with easy to operate push button control.
Precision-built instruments that meet your unique requirements and provide years of trouble free performance in a variety of applications from marine navigation systems; GPS/GLONASS position reporting units as well as rugged hand held RF transceivers for military communication use... or simply an instrument to measure airflows on the job site.
From small compact portable instruments, through larger floor standing models with internal fan-cooled cooling circuits; our experience spans over 30 years of building reliable products used around the world

main: mem per token = 14434244 bytes
main:     load time =  3713.36 ms
main:   sample time =   111.94 ms
main:  predict time = 35878.31 ms / 278.13 ms per token
main:    total time = 40518.79 ms
```

当然也可以使用交互式运行

```
# run the inference with Interactive mode
./main -m ./models/7B/ggml-model-q4_0.bin -t 8 -n 128 --repeat_penalty 1.0 --color -i -r "User:" -p "Transcript of a dialog, where the User interacts with an Assistant named Bob. Bob is helpful, kind, honest, good at writing, and never fails to answer the User's requests immediately and with precision.

User: Hello, Bob.
Bob: Hello. How may I help you today?
User: Please tell me the largest city in Europe.
Bob: Sure. The largest city in Europe is Moscow, the capital of Russia.
User:"
```

```
main: seed = 1678767438
llama_model_load: loading model from './models/7B/ggml-model-q4_0.bin' - please wait ...
llama_model_load: n_vocab = 32000
llama_model_load: n_ctx   = 512
llama_model_load: n_embd  = 4096
llama_model_load: n_mult  = 256
llama_model_load: n_head  = 32
llama_model_load: n_layer = 32
llama_model_load: n_rot   = 128
llama_model_load: f16     = 2
llama_model_load: n_ff    = 11008
llama_model_load: n_parts = 1
llama_model_load: ggml ctx size = 4529.34 MB
llama_model_load: memory_size =   512.00 MB, n_mem = 16384
llama_model_load: loading model part 1/1 from './models/7B/ggml-model-q4_0.bin'
llama_model_load: .................................... done
llama_model_load: model size =  4017.27 MB / num tensors = 291

main: prompt: 'Transcript of a dialog, where the User interacts with an Assistant named Bob. Bob is helpful, kind, honest, good at writing, and never fails to answer the User's requests immediately and with precision.
User: Hello, Bob.
Bob: Hello. How may I help you today?
User: Please tell me the largest city in Europe.
Bob: Sure. The largest city in Europe is Moscow, the capital of Russia.
User:'
main: number of tokens in prompt = 98
     1 -> ''
  4300 -> 'Trans'
   924 -> 'cript'
   310 -> ' of'
   263 -> ' a'
  7928 -> ' dialog'
 29892 -> ','
   988 -> ' where'
   278 -> ' the'
  4911 -> ' User'
 16254 -> ' interact'
 29879 -> 's'
   411 -> ' with'
   385 -> ' an'
  4007 -> ' Ass'
 22137 -> 'istant'
  4257 -> ' named'
  7991 -> ' Bob'
 29889 -> '.'
  7991 -> ' Bob'
   338 -> ' is'
  8444 -> ' helpful'
 29892 -> ','
  2924 -> ' kind'
 29892 -> ','
 15993 -> ' honest'
 29892 -> ','
  1781 -> ' good'
   472 -> ' at'
  5007 -> ' writing'
 29892 -> ','
   322 -> ' and'
  2360 -> ' never'
  8465 -> ' fails'
   304 -> ' to'
  1234 -> ' answer'
   278 -> ' the'
  4911 -> ' User'
 29915 -> '''
 29879 -> 's'
  7274 -> ' requests'
  7389 -> ' immediately'
   322 -> ' and'
   411 -> ' with'
 16716 -> ' precision'
 29889 -> '.'
    13 -> '
'
  2659 -> 'User'
 29901 -> ':'
 15043 -> ' Hello'
 29892 -> ','
  7991 -> ' Bob'
 29889 -> '.'
    13 -> '
'
 29362 -> 'Bob'
 29901 -> ':'
 15043 -> ' Hello'
 29889 -> '.'
  1128 -> ' How'
  1122 -> ' may'
   306 -> ' I'
  1371 -> ' help'
   366 -> ' you'
  9826 -> ' today'
 29973 -> '?'
    13 -> '
'
  2659 -> 'User'
 29901 -> ':'
  3529 -> ' Please'
  2649 -> ' tell'
   592 -> ' me'
   278 -> ' the'
 10150 -> ' largest'
  4272 -> ' city'
   297 -> ' in'
  4092 -> ' Europe'
 29889 -> '.'
    13 -> '
'
 29362 -> 'Bob'
 29901 -> ':'
 18585 -> ' Sure'
 29889 -> '.'
   450 -> ' The'
 10150 -> ' largest'
  4272 -> ' city'
   297 -> ' in'
  4092 -> ' Europe'
   338 -> ' is'
 25820 -> ' Moscow'
 29892 -> ','
   278 -> ' the'
  7483 -> ' capital'
   310 -> ' of'
 12710 -> ' Russia'
 29889 -> '.'
    13 -> '
'
  2659 -> 'User'
 29901 -> ':'

main: interactive mode on.
main: reverse prompt: 'User:'
main: number of tokens in reverse prompt = 2
  2659 -> 'User'
 29901 -> ':'

sampling parameters: temp = 0.800000, top_k = 40, top_p = 0.950000, repeat_last_n = 64, repeat_penalty = 1.000000


== Running in interactive mode. ==
 - Press Ctrl+C to interject at any time.
 - Press Return to return control to LLaMa.
 - If you want to submit another line, end your input in '\'.
Transcript of a dialog, where the User interacts with an Assistant named Bob. Bob is helpful, kind, honest, good at writing, and never fails to answer the User's requests immediately and with precision.
User: Hello, Bob.
Bob: Hello. How may I help you today?
User: Please tell me the largest city in Europe.
Bob: Sure. The largest city in Europe is Moscow, the capital of Russia.
User:
```

这个User后面就可以进行提问，与LLaMA进行交互了

感谢：https://mp.weixin.qq.com/s/OjtjIVTNiXbDA1wTao4JVQ