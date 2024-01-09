<!--⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.
-->

# 从Hub下载文件

`huggingface_hub` 库提供了从存储在 Hub 上的仓库中下载文件的功能。您可以独立使用这些功能，也可以将其集成到自己的库中，使用户更方便地与 Hub 进行交互。本指南将向您展示如何："

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

默认情况下，将下载`main`分支上的最新版本。但是，在某些情况下，您可能希望下载特定版本的文件（例如，从特定分支、PR、标签或提交哈希）。要实现这一点，请使用`revision`参数：

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

要下载整个仓库，只需传递`repo_id`和`repo_type`参数：

运行代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp")
'/home/lysandre/.cache/huggingface/hub/models--lysandre--arxiv-nlp/snapshots/894a9adde21d9a3e3843e6d5aeaaf01875c7fade'

# Or from a dataset
>>> snapshot_download(repo_id="google/fleurs", repo_type="dataset")
'/home/lysandre/.cache/huggingface/hub/datasets--google--fleurs/snapshots/199e4ae37915137c555b1765c01477c216287d34'
```

[`snapshot_download`] 默认下载最新修订版本。如果您想要特定的仓库修订版本，请使用`revision`参数：

运行代码如下：

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", revision="refs/pr/1")
```

### Filter files to download

[`snapshot_download`] provides an easy way to download a repository. However, you don't always want to download the
entire content of a repository. For example, you might want to prevent downloading all `.bin` files if you know you'll
only use the `.safetensors` weights. You can do that using `allow_patterns` and `ignore_patterns` parameters.

These parameters accept either a single pattern or a list of patterns. Patterns are Standard Wildcards (globbing
patterns) as documented [here](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm). The pattern matching is
based on [`fnmatch`](https://docs.python.org/3/library/fnmatch.html).

For example, you can use `allow_patterns` to only download JSON configuration files:

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", allow_patterns="*.json")
```

On the other hand, `ignore_patterns` can exclude certain files from being downloaded. The
following example ignores the `.msgpack` and `.h5` file extensions:

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="lysandre/arxiv-nlp", ignore_patterns=["*.msgpack", "*.h5"])
```

Finally, you can combine both to precisely filter your download. Here is an example to download all json and markdown
files except `vocab.json`.

```python
>>> from huggingface_hub import snapshot_download
>>> snapshot_download(repo_id="gpt2", allow_patterns=["*.md", "*.json"], ignore_patterns="vocab.json")
```

## Download file(s) to local folder

The recommended (and default) way to download files from the Hub is to use the [cache-system](./manage-cache).
You can define your cache location by setting `cache_dir` parameter (both in [`hf_hub_download`] and [`snapshot_download`]).

However, in some cases you want to download files and move them to a specific folder. This is useful to get a workflow
closer to what `git` commands offer. You can do that using the `local_dir` and `local_dir_use_symlinks` parameters:
- `local_dir` must be a path to a folder on your system. The downloaded files will keep the same file structure as in the
repo. For example if `filename="data/train.csv"` and `local_dir="path/to/folder"`, then the returned filepath will be
`"path/to/folder/data/train.csv"`.
- `local_dir_use_symlinks` defines how the file must be saved in your local folder.
  - The default behavior (`"auto"`) is to duplicate small files (<5MB) and use symlinks for bigger files. Symlinks allow
    to optimize both bandwidth and disk usage. However manually editing a symlinked file might corrupt the cache, hence
    the duplication for small files. The 5MB threshold can be configured with the `HF_HUB_LOCAL_DIR_AUTO_SYMLINK_THRESHOLD`
    environment variable.
  - If `local_dir_use_symlinks=True` is set, all files are symlinked for an optimal disk space optimization. This is
    for example useful when downloading a huge dataset with thousands of small files.
  - Finally, if you don't want symlinks at all you can disable them (`local_dir_use_symlinks=False`). The cache directory
    will still be used to check wether the file is already cached or not. If already cached, the file is **duplicated**
    from the cache (i.e. saves bandwidth but increases disk usage). If the file is not already cached, it will be
    downloaded and moved directly to the local dir. This means that if you need to reuse it somewhere else later, it
    will be **re-downloaded**.

Here is a table that summarizes the different options to help you choose the parameters that best suit your use case.

<!-- Generated with https://www.tablesgenerator.com/markdown_tables -->
| Parameters | File already cached | Returned path | Can read path? | Can save to path? | Optimized bandwidth | Optimized disk usage |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `local_dir=None` |  | symlink in cache | ✅ | ❌<br>_(save would corrupt the cache)_ | ✅ | ✅ |
| `local_dir="path/to/folder"`<br>`local_dir_use_symlinks="auto"` |  | file or symlink in folder | ✅ | ✅ _(for small files)_ <br> ⚠️ _(for big files do not resolve path before saving)_ | ✅ | ✅ |
| `local_dir="path/to/folder"`<br>`local_dir_use_symlinks=True` |  | symlink in folder | ✅ | ⚠️<br>_(do not resolve path before saving)_ | ✅ | ✅ |
| `local_dir="path/to/folder"`<br>`local_dir_use_symlinks=False` | No | file in folder | ✅ | ✅ | ❌<br>_(if re-run, file is re-downloaded)_ | ⚠️<br>(multiple copies if ran in multiple folders) |
| `local_dir="path/to/folder"`<br>`local_dir_use_symlinks=False` | Yes | file in folder | ✅ | ✅ | ⚠️<br>_(file has to be cached first)_ | ❌<br>_(file is duplicated)_ |

**Note:** if you are on a Windows machine, you need to enable developer mode or run `huggingface_hub` as admin to enable
symlinks. Check out the [cache limitations](../guides/manage-cache#limitations) section for more details.

## Download from the CLI

You can use the `huggingface-cli download` command from the terminal to directly download files from the Hub.
Internally, it uses the same [`hf_hub_download`] and [`snapshot_download`] helpers described above and prints the
returned path to the terminal.

```bash
>>> huggingface-cli download gpt2 config.json
/home/wauplin/.cache/huggingface/hub/models--gpt2/snapshots/11c5a3d5811f50298f278a704980280950aedb10/config.json
```

You can download multiple files at once which displays a progress bar and returns the snapshot path in which the files
are located:

```bash
>>> huggingface-cli download gpt2 config.json model.safetensors
Fetching 2 files: 100%|████████████████████████████████████████████| 2/2 [00:00<00:00, 23831.27it/s]
/home/wauplin/.cache/huggingface/hub/models--gpt2/snapshots/11c5a3d5811f50298f278a704980280950aedb10
```

For more details about the CLI download command, please refer to the [CLI guide](./cli#huggingface-cli-download).

## Faster downloads

If you are running on a machine with high bandwidth, you can increase your download speed with [`hf_transfer`](https://github.com/huggingface/hf_transfer), a Rust-based library developed to speed up file transfers with the Hub. To enable it, install the package (`pip install hf_transfer`) and set `HF_HUB_ENABLE_HF_TRANSFER=1` as an environment variable.

<Tip>

Progress bars are supported in `hf_transfer` starting from version `0.1.4`. Consider upgrading (`pip install -U hf-transfer`) if you plan to enable faster downloads.

</Tip>

<Tip warning={true}>

`hf_transfer` is a power user tool! It is tested and production-ready, but it lacks user-friendly features like advanced error handling or proxies. For more details, please take a look at this [section](https://huggingface.co/docs/huggingface_hub/hf_transfer).

</Tip>