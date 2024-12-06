---
title: 在VSCode中使用远程多卡 Ollama
last_modified_at: 2024-11-30 12:34:56 +0000
---
# 在VSCode中使用远程多卡 Ollama

对于开发大模型语言的同学们，都想使用高性能显卡来跑模型，进行调优或者推理，但是如果个人购买一张或者多张显卡的话，成本还是很高的，当然，土豪除外。

本例就是利用 算力平台租借显卡，来运行大模型，然后将其设置成为模型服务器，在本地可以进行调用，支持开发工作。




Ollama 可以非常方便的运行大语言模型，在本地或者算力平台上部署后，可以做各类AI应用的原型开发。

这里介绍一下在 [算力平台 AutoDL](https://www.autodl.com/home)上部署 Ollama，然后在本地VSCode中，如何使用该 Ollama服务的。

为了更强的推理效果，本利使用了两张 4090显卡，基础磁盘空间扩大了 100GB，模型使用了 qwen2.5-coder:32b

---

## 创建基础环境

**多卡选择**：

AutoDL上可以根据需要选择不同类型的显卡组合，本次尝试使用两张 4090显卡，将默认磁盘扩充 100GB空间，确保模型文件有足够空间存放。

![](/experience/assets/images/posts/ai/ollama/autodl_choose_gpu.png)


系统镜像直接使用了 agiclass 提供的 vllm部署镜像，可以另外实验 vllm的部署。本例中主要验证 Ollama的部署。 或者直接使用 Ollama/ollama 镜像，该镜像补充了一些系统库，可以支持 api/generate 接口。

![](/experience/assets/images/posts/ai/ollama/autodl_choose_model.png)

### 安装Ollama

创建基础环境后，ssh 登录，直接运行命令行安装。详细可参考 [Ollama官方文档](https://ollama.com/download/linux)

```
curl -fsSL https://ollama.com/install.sh | sh
```


启动 Ollama服务，最后两行清楚的看到了， Ollama加载了两张4090的显卡，非常方便，不用做任何特殊设置。如果需要后台守护进程，可以使用 'tmux' 命令，将 Ollama服务后台运行。


```
apt-get update && apt-get install -y tmux
tmux


ollama serve
2024/11/30 12:03:38 routes.go:1197: INFO server config env="map[CUDA_VISIBLE_DEVICES: GPU_DEVICE_ORDINAL: HIP_VISIBLE_DEVICES: HSA_OVERRIDE_GFX_VERSION: HTTPS_PROXY: HTTP_PROXY: NO_PROXY: OLLAMA_DEBUG:false OLLAMA_FLASH_ATTENTION:false OLLAMA_GPU_OVERHEAD:0 OLLAMA_HOST:http://127.0.0.1:11434 OLLAMA_INTEL_GPU:false OLLAMA_KEEP_ALIVE:5m0s OLLAMA_LLM_LIBRARY: OLLAMA_LOAD_TIMEOUT:5m0s OLLAMA_MAX_LOADED_MODELS:0 OLLAMA_MAX_QUEUE:512 OLLAMA_MODELS:/root/autodl-tmp/ollama_images/ OLLAMA_MULTIUSER_CACHE:false OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:0 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://* vscode-webview://*] OLLAMA_SCHED_SPREAD:false OLLAMA_TMPDIR: ROCR_VISIBLE_DEVICES: http_proxy: https_proxy: no_proxy:]"
time=2024-11-30T12:03:38.994+08:00 level=INFO source=images.go:753 msg="total blobs: 0"
time=2024-11-30T12:03:38.994+08:00 level=INFO source=images.go:760 msg="total unused blobs removed: 0"
time=2024-11-30T12:03:38.994+08:00 level=INFO source=routes.go:1248 msg="Listening on 127.0.0.1:11434 (version 0.4.6)"
time=2024-11-30T12:03:39.006+08:00 level=INFO source=common.go:135 msg="extracting embedded files" dir=/tmp/ollama3115970198/runners
time=2024-11-30T12:03:39.148+08:00 level=INFO source=common.go:49 msg="Dynamic LLM libraries" runners="[cpu_avx cpu_avx2 cuda_v11 cuda_v12 rocm cpu]"
time=2024-11-30T12:03:39.148+08:00 level=INFO source=gpu.go:221 msg="looking for compatible GPUs"
time=2024-11-30T12:03:39.478+08:00 level=INFO source=types.go:123 msg="inference compute" id=GPU-227e3a6e-2b8e-0415-3503-e856c0b4b681 library=cuda variant=v12 compute=8.9 driver=12.4 name="NVIDIA GeForce RTX 4090" total="23.6 GiB" available="23.3 GiB"
time=2024-11-30T12:03:39.478+08:00 level=INFO source=types.go:123 msg="inference compute" id=GPU-80dd7e0f-c9d4-ffa7-7ebc-88e5d2420202 library=cuda variant=v12 compute=8.9 driver=12.4 name="NVIDIA GeForce RTX 4090" total="23.6 GiB" available="23.3 GiB"
```


另外开一个 ssh 窗口，下载模型。上面的窗口，可以查看 Ollama server 的动态日志。
设置模型保存路径，系统盘空间有限。autodl-tmp 是外挂盘的路径。

```
mkdir /root/autodl-tmp/ollama_images
export OLLAMA_MODELS="/root/autodl-tmp/ollama_images/"

ollama run qwen2.5-coder:32b
```



### 验证Ollama运行状态

直接和 Ollama进行交互，生成一个 Dockerfile

![](/experience/assets/images/posts/ai/ollama/talk_with_ollama_qwen2.5.png)

### 导出Ollama服务端口到本地

为了能够在本机访问直接访问到这个 Ollama服务，需要将这个台远程机器的Ollama服务端口映射到本地， AutoDL提供了 SSH Tunnel功能，将远程机器的端口映射到本地。

![](/experience/assets/images/posts/ai/ollama/enable_ssh_tolocalport.png)

样例中，是将 6006端口映射到本地 6006端口，而Ollama服务端口是 11434。所以，我们需要修改命令如下。

在本地命令行窗口中执行该命令，密码仍然是AutoDL提供的密码。执行后，没有任何提示，说明映射成功。
```
ssh -CNg -L 11434:127.0.0.1:11434 root@connect.westb.seetacloud.com -p 13040
```

### 本地验证 Ollama服务

打开本地浏览器，输入地址 "localhost:11434"，可以看到“Ollama is running”

![](/experience/assets/images/posts/ai/ollama/localhost_verify_ollama.png)

同时，查看 Ollama的服务窗口，可以看到有消息发送到服务端，说明 Ollama服务已经可以在本地进行访问调用了。

![](/experience/assets/images/posts/ai/ollama/ollama_service_well.png)

## 设置 VSCode Continue

Continue 插件使用默认设置，自动发现服务。

![](/experience/assets/images/posts/ai/ollama/continue_default_configuration.png)

打开一个项目文件，可以看到 Continue已经通过 “localhost:11434” 找到了 qwen2.5-coder:32b


![](/experience/assets/images/posts/ai/ollama/continue_find_ollama.png)

让 Continue优化一下当前打开的 Dockerfile，验证整体过程。

![](/experience/assets/images/posts/ai/ollama/continue_refine_dockerfile.png)

---

## 序列图[序列图](https://github.com/knsv/mermaid#sequence-diagram)

<div class="mermaid">
sequenceDiagram
  participant VSCode
  participant SSH
  VSCode->>Ollama: Please help me to refine the code?
  loop thinking
    Ollama->>Ollama: generate the code
  end
  Note right of Ollama: referring history context...
  Ollama-->>VSCode: Code suggestion for you!
</div>

