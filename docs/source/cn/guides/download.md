<!--⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.
-->

# 从Hub下载文件

`huggingface_hub` 库提供了从存储在 Hub 上的仓库中下载文件的功能。您可以独立使用这些功能，也可以将其集成到自己的库中，使用户更方便地与 Hub 进行交互。本指南将向您展示如何：

* 下载并缓存单个文件
* 下载并缓存整个存储库
* 将文件下载到本地文件夹

## 下载单个文件

`hf_hub_download` 函数是从 Hub 下载文件的主要函数。它会下载远程文件，在磁盘上进行缓存（以版本感知的方式），并返回本地文件路径。

<Tip>

返回的文件路径是指向 HF 本地缓存的指针。因此，重要的是不要修改文件，以避免破坏缓存。如果您想了解有关文件如何被缓存的更多信息，请参阅我们的[缓存指南](./manage-cache)。

</Tip>

### 从最新版本

使用 `repo_id`、`repo_type` 和 `filename` 参数选择要下载的文件。默认情况下，该文件将被视为属于 `model` 仓库的一部分。

请运行以下代码：

```python
>>> from huggingface_hub import hf_hub_download
>>> hf_hub_download(repo_id="lysandre/arxiv-nlp", filename="config.json")
'/root/.cache/huggingface/hub/models--lysandre--arxiv-nlp/snapshots/894a9adde21d9a3e3843e6d5aeaaf01875c7fade/config.json'

# Download from a dataset
>>> hf_hub_download(repo_id="google/fleurs", filename="fleurs.py", repo_type="dataset")
'/root/.cache/huggingface/hub/datasets--google--fleurs/snapshots/199e4ae37915137c555b1765c01477c216287d34/fleurs.py'
```

### 从特定版本

默认情况下，将下载 `main`分支上的最新版本。但是，在某些情况下，您可能希望下载特定版本的文件（例如，从特定分支、PR、标签或提交哈希）。要实现这一点，请使用 `revision`参数：

请运行以下代码：

```python
# Download from the `v1.0` tag
>>> hf_hub_download(repo_id="lysandre/arxiv-nlp", filename="config.json", revision="v1.0")

# Download from the `test-branch` branch
>>> hf_hub_download(repo_id="lysandre/arxiv-nlp", filename="config.json", revision="test-branch")

# Download from Pull Request #3
>>> hf_hub_download(repo_id="lysandre/arxiv-nlp", filename="config.json", revision="refs/pr/3")

# Download from a specific commit hash
>>> hf_hub_download(repo_id="lysandre/arxiv-nlp", filename="config.json", revision="877b84a8f93f2d619faa2a6e514a32beef88ab0a")
```

**笔记:** 在使用提交哈希时，必须使用完整长度的哈希，而不是7个字符的提交哈希。

### 构建下载URL

如果您想构建用于从仓库下载文件的URL，可以使用 [`hf_hub_url`]，该函数返回一个URL。请注意，它在 [`hf_hub_download`] 中被内部使用。

## 下载整个仓库

[`snapshot_download`] 在给定的修订版本中下载整个仓库。它在内部使用 [`hf_hub_download`]，这意味着所有下载的文件也会被缓存在您的本地磁盘上。下载是多线程进行的，以加快进程。

要下载整个仓库，只需传递 `repo_id`和 `repo_type`参数：

运行代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp")
'/home/lysandre/.cache/huggingface/hub/models--lysandre--arxiv-nlp/snapshots/894a9adde21d9a3e3843e6d5aeaaf01875c7fade'

# Or from a dataset
>>> snapshot_download(repo_id="google/fleurs", repo_type="dataset")
'/home/lysandre/.cache/huggingface/hub/datasets--google--fleurs/snapshots/199e4ae37915137c555b1765c01477c216287d34'
```

[`snapshot_download`] 默认下载最新修订版本。如果您想要特定的仓库修订版本，请使用 `revision`参数：

运行代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", revision="refs/pr/1")
```

### 筛选要下载的文件

[`snapshot_download`] 提供了一个简便的方法来下载一个仓库。然而，我们并不总是希望下载整个仓库的内容。例如，如果只需要使用 `.safetensors` 权重文件，你可能希望阻止下载所有的 `.bin` 文件。你可以使用 `allow_patterns` 和 `ignore_patterns` 这两个参数来实现这个目的。

这些参数接受单个模式或模式列表。这些模式是标准通配符（globbing patterns），在[这里](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm)有文档说明。模式匹配基于 [`fnmatch`](https://docs.python.org/3/library/fnmatch.html)。

例如，你可以使用 `allow_patterns` 来只下载 JSON 配置文件：

运行代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", allow_patterns="*.json")
```

另一方面，`ignore_patterns` 可以排除某些文件不被下载。下面的示例忽略了 `.msgpack` 和 `.h5` 文件扩展名：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", ignore_patterns=["*.msgpack", "*.h5"])
```

最后，您可以结合两者精确过滤您的下载。以下是一个示例，用于下载所有 json 和 markdown 文件，除了 `vocab.json`。

代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="gpt2", allow_patterns=["*.md", "*.json"], ignore_patterns="vocab.json")
```

## 将文件下载到本地文件夹

从Hub下载文件的推荐方式（也是默认方式）是使用[缓存系统](./manage-cache)。
您可以通过设置 `cache_dir` 参数（在 [`hf_hub_download`] 和 [`snapshot_download`] 中均可）来定义缓存位置。

然而，在某些情况下，您希望下载文件并将它们移动到特定的文件夹中。这对于建立工作流程很有用。
您可以使用 `local_dir` 和 `local_dir_use_symlinks` 参数来实现这一点，使其更接近 `git` 命令提供的功能:

- `local_dir` 必须是系统上文件夹的路径。下载的文件将保持与存储库中相同的文件结构。例如，如果 `filename="data/train.csv"`，`local_dir="path/to/folder"`，那么返回的文件路径将是 `"path/to/folder/data/train.csv"`。
- `local_dir_use_symlinks` 定义了文件在本地文件夹中的保存方式。
  - 默认行为（`"auto"`）是对小文件（<5MB）进行复制，并对大文件使用符号链接。符号链接可以优化带宽和磁盘使用。但手动编辑符号链接的文件可能会损坏缓存，因此对于小文件进行复制。5MB 阈值可以通过环境变量 `HF_HUB_LOCAL_DIR_AUTO_SYMLINK_THRESHOLD` 进行配置。
  - 如果设置 `local_dir_use_symlinks=True`，则所有文件都将使用符号链接进行磁盘空间优化。例如，在下载包含数千个小文件的大型数据集时非常有用。
  - 最后，如果您完全不想使用符号链接，可以禁用它们（`local_dir_use_symlinks=False`）。仍然将使用缓存目录来检查文件是否已经缓存。如果已经缓存，文件将从缓存中**复制**（即节省带宽但增加磁盘使用量）。如果文件尚未缓存，它将直接下载并移动到本地目录。这意味着如果以后需要在其他地方重用它，它将被**重新下载**。

以下是一个总结不同选项的表格，以帮助您选择最适合您用例的参数。

<!-- Generated with https://www.tablesgenerator.com/markdown_tables -->

| Parameters                                                          | File already cached |       Returned path       | Can read path? |                                      Can save to path?                                      |               Optimized bandwidth               |                   Optimized disk usage                   |
| ------------------------------------------------------------------- | :-----------------: | :-----------------------: | :------------: | :-----------------------------------------------------------------------------------------: | :----------------------------------------------: | :------------------------------------------------------: |
| `local_dir=None`                                                  |                    |     symlink in cache     |       ✅       |                        ❌`<br>`_(save would corrupt the cache)_                        |                        ✅                        |                            ✅                            |
| `local_dir="path/to/folder"<br>``local_dir_use_symlinks="auto"` |                    | file or symlink in folder |       ✅       | ✅_(for small files)_ `<br>` ⚠️ _(for big files do not resolve path before saving)_ |                        ✅                        |                            ✅                            |
| `local_dir="path/to/folder"<br>``local_dir_use_symlinks=True`   |                    |     symlink in folder     |       ✅       |                     ⚠️`<br>`_(do not resolve path before saving)_                     |                        ✅                        |                            ✅                            |
| `local_dir="path/to/folder"<br>``local_dir_use_symlinks=False`  |         No         |      file in folder      |       ✅       |                                             ✅                                             | ❌`<br>`_(if re-run, file is re-downloaded)_ | ⚠️`<br>`(multiple copies if ran in multiple folders) |
| `local_dir="path/to/folder"<br>``local_dir_use_symlinks=False`  |         Yes         |      file in folder      |       ✅       |                                             ✅                                             |  ⚠️`<br>`_(file has to be cached first)_  |            ❌`<br>`_(file is duplicated)_            |

**注意** 如果您正在使用 Windows 机器，您需要启用开发者模式或以管理员身份运行 `huggingface_hub` 以启用符号链接。更多详细信息，请查看 [缓存限制](../guides/manage-cache#limitations) 部分。

## 从命令行界面下载

您可以使用终端中的 `huggingface-cli download` 命令直接从 Hub 下载文件。
在内部，它使用了上述描述的相同 [hf_hub_download] 和 [snapshot_download] 辅助工具，并将返回的路径打印到终端上。

代码如下：

```bash
>>> huggingface-cli download gpt2 config.json
/home/wauplin/.cache/huggingface/hub/models--gpt2/snapshots/11c5a3d5811f50298f278a704980280950aedb10/config.json
```

您可以一次下载多个文件，显示进度条，并返回文件所在的快照路径：

代码如下：

```bash
>>> huggingface-cli download gpt2 config.json model.safetensors
Fetching 2 files: 100%|████████████████████████████████████████████| 2/2 [00:00<00:00, 23831.27it/s]
/home/wauplin/.cache/huggingface/hub/models--gpt2/snapshots/11c5a3d5811f50298f278a704980280950aedb10
```

有关 CLI 下载命令的更多详细信息，请参阅 [CLI 指南](./cli#huggingface-cli-download)。

## 更快的下载

如果您在带宽较高的计算机上运行，您可以通过 [`hf_transfer`](https://github.com/huggingface/hf_transfer) 来提高下载速度，这是一个基于 Rust 开发的库，专门用于加速与 Hub 的文件传输。要启用它，请安装该软件包（`pip install hf_transfer`）并将 `HF_HUB_ENABLE_HF_TRANSFER=1` 设置为环境变量。

<Tip>

从版本 `0.1.4` 开始，`hf_transfer` 支持进度条显示。如果您计划启用更快的下载，请考虑升级（`pip install -U hf-transfer`）。

</Tip>

<Tip warning={true}>

`hf_transfer` 是一款高级用户工具！它经过测试并准备投入生产，但缺少用户友好的功能，如高级错误处理或代理设置。有关更多详细信息，请查看这个[部分](https://huggingface.co/docs/huggingface_hub/hf_transfer)。

</Tip>
