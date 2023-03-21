# Web Stable Diffusion_zh

**原文链接**：https://zhuanlan.zhihu.com/p/612517660

我们在最近见证了 AI 模型所取得的惊人的巨大进展。归功于整个生态系统的开源努力，开发者们现在可以轻松地将各种开源模型组合在一起以应对各种机器学习任务。Stable diffusion 模型能够基于文本输入自动生成各种风格迥异的图片。这些模型通常具有很大的规模和庞大的计算量，意味着当我们尝试基于这些模型开发网页应用时，我们必须将所有的计算交给 (GPU) 服务器。此外，大多数此类计算必须运行在特定类型的、流行深度学习框架已经能够成熟支持的 GPU 上。

我们是否可以改变这一现状呢？我们有许多将一部分或者全部的计算交由 client 来完成的理由。这样做有很多可能的好处，比如节约租用服务器的费用，同时也能够在模型个性化和隐私保护等方面有所提升。PC 和移动设备正在朝着使这些好处成为可能的方向发展，client 端在逐渐变得相当强大。比如，最新的 MacBook Pro 可以拥有最高 96GB、能够轻松存下全部模型参数的统一内存，同时拥有能够运行许多模型的相当强大的 GPU。

尽管可以为这些模型提供特定的 client 应用来运行它们（我们确实同样能够支持这一点），“只需在浏览器里打开一个标签页，就能够完全在本地运行这些模型” 这件事情，是不是听起来更具吸引力呢？放眼当前，整个生态系统在一定程度上已经做好了准备：WebAssembly 允许将更多底层 runtime 接入网页端；为了加速计算，在近段时间越来越成熟的 WebGPU 让我们能够在浏览器里直接进行本地 GPU 的执行。

在本文中我们将给大家介绍 Web Stable Diffusion。这是世界上的第一个通过深度学习编译技术将 stable diffusion 完全运行在浏览器中的项目。**模型的全部一切都运行在浏览器里，无需云端服务器支持。**

![img](https://pic2.zhimg.com/80/v2-22cd914a66185ad1b33cb73dc9ef4099_720w.webp)

Web Stable Diffusion demo

其实，无论是在硬件层面还是浏览器生态系统层面，我们已经初步集成了这些必要的要素。然而，想让模型真正在浏览器里跑起来，还有解决很多关键的问题需要我们去解决：

- 需要在没有具备 GPU 加速的 Python 框架的地方部署模型；
- 绝大多数 AI 框架都对硬件厂商已经优化过的计算库有很大的依赖，而我们需要从零开始。为了在不同的 client 端都能部署，需要适应不同的 client 环境；
- 为了让整个模型能够被放入内存中，需要精心对内存使用进行规划。

我们并不只想局限于 stable diffusion 模型。相对地，我们想要提供一个能够重复使用的、便于快速开发的、可组合的工作流，从而让每个人都能够轻松地在 **Python-first** 的环境中开发并优化这些模型，同时能够将这些模型**更通用地部署**在每个角落，包括网页。

## 怎么做？

这里的关键技术是机器学习编译 (MLC)。我们的解决方案构建在开源生态系统的基础之上，这其中包括 PyTorch 框架， Hugging Face diffusers 库和 tokenizers 库，rust, wasm 和 WebGPU。本工作的核心技术基于 Apache TVM Unity，它是 [Apache TVM 社区](https://link.zhihu.com/?target=https%3A//tvm.apache.org/)中正在持续推进的一个激动人心的方向。

- 使用 Hugging Face diffusers 库中的 [runwayml/stable-diffusion-v1-5](https://link.zhihu.com/?target=https%3A//huggingface.co/runwayml/stable-diffusion-v1-5/tree/main) 模型；
- 通过 [TorchDynamo](https://link.zhihu.com/?target=https%3A//pytorch.org/tutorials/intermediate/dynamo_tutorial.html) 和 [Torch FX](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/fx.html) 获取模型中的计算和关键部分，打包成为 TVM 中的一个 IRModule；
- TVM IRModule 内部的每一个函数都能够被进一步地优化和变换，并最终生成能够通用部署在支持 minimum TVM runtime 的后端环境上 (JavaScript 就是其中之一)。
- [TensorIR](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2207.04296) 和 [MetaSchedule](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2205.13603) 可以自动生成能够优化模型中每个子程序的程序变换。我们在特定的设备上使用本地的 GPU runtime 进行一定的搜索，从而获得这些程序变换。我们基于这些变换生成优化后的 GPU shader。我们提供一个存有这些变换的数据库，这使得每一次重新构建模型时可以直接获得能够优化程序的最佳变换。
- 搭建静态的内存规划，从而能够实现模型不同层之间的内存复用，减少大量的内存开销。
- 使用 [Emscripten](https://link.zhihu.com/?target=https%3A//emscripten.org/) 和 TypeScript 搭建了能够部署模型的 TVM web runtime。
- 使用 Hugging Face [Rust tokenizers 库](https://link.zhihu.com/?target=https%3A//github.com/huggingface/tokenizers)中的 [wasm 接口](https://link.zhihu.com/?target=https%3A//blog.mithrilsecurity.io/porting-tokenizers-to-wasm/)。

![img](https://pic3.zhimg.com/80/v2-e1aa098292e4bd124dc1e2f6f91691fa_720w.webp)

Web Stable Diffusion 的工作流

除了最后有一段将所有部分连接在一起的 400 行 JavaScript，工作流中的所有步骤都在 Python 中完成。在 Python 中这样交互开发各种新模型的过程十分有趣。

所有的这些都是依靠各个开源生态系统才成为可能。本工作大量使用了 [Apache TVM Unity](https://link.zhihu.com/?target=https%3A//discuss.tvm.apache.org/t/establish-tvm-unity-connection-a-technical-strategy/13344) —— TVM 项目中最新的方向。TVM Unity 能够使机器学习编译的开发体验全部围绕 Python 这一机器学习生态中每个人所熟知的语言展开。TVM Unity 使得我们能够轻松地在 Python 中将各个层级、各种角度的优化组合在一起，逐渐将 stable diffusion 应用带至网页端。TVM Unity 同时为机器学习编译的生态系统提供了将多种新的解决方案组装在一起的便利，比如，我们在未来可以简单地将其它 WebGPU shader 或者 shader libraries 引入到工作流中。

## 与本地 GPU Runtime 的比较，局限性与可能性

除了 WebGPU runtime，这个项目同时提供了将 stable diffusion 模型部署在本地 GPU runtime 的选项（会在近日更新至 [GitHub 仓库](https://link.zhihu.com/?target=https%3A//github.com/mlc-ai/web-stable-diffusion%23get-started)）。这些选项不仅能够用于在本地环境部署模型，还可以作为 WebGPU 和本地 GPU 的性能比较的一个参考。

从原理上讲，WebGPU 通过将 WGSL (WebGPU Shading Language) shader 翻译为本地 GPU shader 的方式运作。所以，理论上我们可以将在 WebGPU 中执行与在本地 GPU 的执行的性能差距缩小为零。然而，如果直接在 Apple silicon 驱动的 MacBook 上使用 Chrome 运行现在的[网页 demo](https://link.zhihu.com/?target=https%3A//mlc.ai/web-stable-diffusion/)，我们会发现 WebGPU 相比本地 GPU 有大约三倍的性能退化。这是因为 Chrome 的 WebGPU 实现向所有的数组访问都插入了边界检查（比如 `a[i]` 会变为 `a[min(i, a.size)]`。在理想情况中，下游 shader 的编译器需要能够进行相应优化，去除这类裁切。然而目前这一优化还没有成熟。随着 WebGPU 的实现逐渐变得成熟，这一性能差距将会消失。

不过，我们可以通过在命令行中使用一个特殊的 flag 启动 Chrome 来绕过这一问题。我们只需要完全退出 Chrome，并在命令行里输入以下命令重新启动 Chrome 即可。

```
/path/to/chrome-canary --enable-dawn-features=disable_robustness
```

通过这个命令启动 Chrome，我们会发现模型在 WebGPU 中的运行速度和在本地 GPU 环境中运行速度基本一致。

我们相信对与机器学习模型更广泛、更通用的部署将成为趋势，而现在这只是一切的开始。WebGPU 仍在演化之中，Chrome 计划在今年五月正式发布 WebGPU，目前 WebGPU 仅在 Chrome Canary 浏览器里可以使用，也具有不稳定性。同时 WebGPU 目前还存在一些其它限制，比如当前仅存在对 FP32 的支持（规范中有 FP16 的 shader 扩展，但目前没有实现）。经过本文工作流优化后的 stable diffusion 模型将会需要有一定 RAM (大约 8GB) 的 GPU。我们目前仅在 Apple silicon 驱动的 MacBook 环境中进行了测试。此外，还存在引入更多高级优化（比如 [FlashAttention](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2205.14135)）和 quantization，进一步提升整个系统性能的可能性。

这些改进空间将为当前解决方案带来若干倍的性能提升，相信上面这些中的很多问题将在不久的将来得到解决。值得注意的是，Web Stable Diffusion 这一解决方案中的单个组成部分也可以非常有用。比如，可以选择只在本地部署模型的 text encoder 这部分。除此之外，这里 Python-first 的开发模式和通用的部署工作流可以将机器学习模型带至其它环境，诸如新兴涌现的硬件或移动设备等。最后，这里的机器学习编译栈同样能够被使用在服务器端，用于优化服务器上的机器学习任务。

## 相关资源

- [网页 demo](https://link.zhihu.com/?target=https%3A//mlc.ai/web-stable-diffusion)
- [GitHub 仓库](https://link.zhihu.com/?target=https%3A//github.com/mlc-ai/web-stable-diffusion)
- [mlc.ai 机器学习编译社区](https://link.zhihu.com/?target=https%3A//mlc.ai/)



