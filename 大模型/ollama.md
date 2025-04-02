# ollama

### 修改提示词长度上限（默认2048）

1. 到处模型的Modelfile

   ```sh
   ollama show --modelfile qwen2.5:32b > Modelfile
   ```

2. 修改Modelfile填写参数

   ```sh
   vim Modelfiel
   ```

   ![image-20250315172146984](E:/Learning/notes/LearningNotes/images/image-20250315172146984.png)

3. 用Modelfile创建新模型

   ```sh
   ollama create qwen2.5:32b-6k -f Modelfile   
   ```

   

### `ollama serve`的核心参数

这是启动ollama服务器的命令，运行大模型或执行ollama其他命令之前，都需要先启动ollama服务器

- ollama启动过程中会识别可用的显卡资源

```sh
czw@84:~/graphrag-test/ragtest$ ollama serve --help
Start ollama

Usage:
  ollama serve [flags]

Aliases:
  serve, start

Flags:
  -h, --help   help for serve

Environment Variables:
      OLLAMA_DEBUG               Show additional debug information (e.g. OLLAMA_DEBUG=1)
      OLLAMA_HOST                IP Address for the ollama server (default 127.0.0.1:11434)
      OLLAMA_KEEP_ALIVE          The duration that models stay loaded in memory (default "5m")
      OLLAMA_MAX_LOADED_MODELS   Maximum number of loaded models per GPU
      OLLAMA_MAX_QUEUE           Maximum number of queued requests
      OLLAMA_MODELS              The path to the models directory
      OLLAMA_NUM_PARALLEL        Maximum number of parallel requests
      OLLAMA_NOPRUNE             Do not prune model blobs on startup
      OLLAMA_ORIGINS             A comma separated list of allowed origins
      OLLAMA_SCHED_SPREAD        Always schedule model across all GPUs
                                 
      OLLAMA_FLASH_ATTENTION     Enabled flash attention
      OLLAMA_KV_CACHE_TYPE       Quantization type for the K/V cache (default: f16)
      OLLAMA_LLM_LIBRARY         Set LLM library to bypass autodetection
      OLLAMA_GPU_OVERHEAD        Reserve a portion of VRAM per GPU (bytes)
      OLLAMA_LOAD_TIMEOUT        How long to allow model loads to stall before giving up (default "5m")
```

---

下面列出了通过 `ollama serve --help` 显示的环境变量、它们的默认值（如果有明确说明）以及各自的含义：

#### 1. 基本环境变量

- **OLLAMA_DEBUG**
  - **默认值**：无默认值（需要时手动设置，例如 `OLLAMA_DEBUG=1`）
  - **含义**：启用后会输出额外的调试信息，有助于诊断问题。
- **OLLAMA_HOST**
  - **默认值**：`127.0.0.1:11434`
  - **含义**：指定 Ollama 服务器监听的 IP 地址和端口。默认的回环地址使得ollama api只能本机上可访问，如果需要允许外部访问，需要设置为`0.0.0.0:11434`。
- **OLLAMA_KEEP_ALIVE**
  - **默认值**：`5m`
  - **含义**：模型在（GPU）内存中保持加载状态的持续时间。也就是说，在模型被调用后，它会在（GPU）内存中保留 5 分钟以便于快速响应后续请求。如果想要模型一直装载在显卡中，可以设置为`-1`。
- **OLLAMA_LOAD_TIMEOUT**
  - **默认值**：`5m`
  - **含义**：在加载模型时，如果超过 5 分钟仍然卡住，就会放弃加载。

#### 2. 资源和任务调度相关

- **OLLAMA_MAX_LOADED_MODELS**
  - **默认值**：1
  - **含义**：每个 GPU 上允许同时加载的最大模型数量，超出后可能会卸载一些模型以节省显存。
- **OLLAMA_MAX_QUEUE**
  - **默认值**：512
  - **含义**：请求队列中允许等待处理的最大请求数，超过这个数时新请求可能会被拒绝或延迟处理。
- **OLLAMA_NUM_PARALLEL**
  - **默认值**：`1`（单线程处理）
  - **含义**：允许同时处理的最大并行请求数。

#### 3. 模型和文件路径

- **OLLAMA_MODELS**
  - **默认值**：未明确给出
  - **含义**：指定存放模型文件的目录路径，Ollama 会从这个目录中加载模型。
- **OLLAMA_NOPRUNE**
  - **默认值**：未明确给出
  - **含义**：如果设置了该变量，启动时不会清理（prune）不再使用的模型 blob。

#### 4. 网络与安全相关

- **OLLAMA_ORIGINS**
  - **默认值**：未明确给出
  - **含义**：允许访问 API 的来源（Origin）的逗号分隔列表，通常用于配置 CORS（跨源资源共享）。
- **OLLAMA_SCHED_SPREAD**
  - **默认值**：未明确给出
  - **含义**：如果设置，则在调度模型时，会尝试将模型均匀分布在所有 GPU 上。

#### 5. 性能与模型推理相关

- **OLLAMA_FLASH_ATTENTION**
  - **默认值**：关闭
  - **含义**：启用后使用 Flash Attention 优化，以提高推理速度（Flash Attention 是一种高效的注意力计算实现）。
- **OLLAMA_KV_CACHE_TYPE**
  - **默认值**：`f16`
  - **含义**：用于保存键值缓存的量化类型。`f16` 表示使用半精度（16 位浮点数），可以节省显存开销。
- **OLLAMA_LLM_LIBRARY**
  - **默认值**：未明确给出
  - **含义**：允许手动指定后端使用的大语言模型库，以绕过自动检测机制。
- **OLLAMA_GPU_OVERHEAD**
  - **默认值**：自动计算
  - **含义**：每个 GPU 保留一部分显存作为预留（以字节为单位），以防止内存耗尽。

------

### 总结

这些环境变量主要用于控制 Ollama 的服务监听、模型加载、请求调度以及性能优化等方面。默认情况下，只有 `OLLAMA_HOST`（`127.0.0.1:11434`）、`OLLAMA_KEEP_ALIVE`（`5m`）、`OLLAMA_LOAD_TIMEOUT`（`5m`）和 `OLLAMA_KV_CACHE_TYPE`（`f16`）等有明确默认值，其余变量如果不设置，则采用系统内部的默认策略。

你可以根据自己的需求通过设置这些环境变量来调整 Ollama 的行为，例如：

```bash
export OLLAMA_HOST="0.0.0.0:11434"   # 让服务监听所有网络接口
export OLLAMA_KEEP_ALIVE="10m"         # 延长模型保持加载的时间
export OLLAMA_GPU_OVERHEAD="100000000" # 预留 100MB 显存
```

通过合理配置这些环境变量，你可以更好地控制和优化 Ollama 服务的运行。

