<!--⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.
-->

# 通过文件系统的API与Hub进行交互

除了 [`HfApi`] 之外，`huggingface_hub` 库还提供了 [`HfFileSystem`]接口，这是一个符合 [fsspec 兼容性](https://filesystem-spec.readthedocs.io/en/latest/) 的 Python 文件接口，用于与 Hugging Face Hub 进行交互。[`HfFileSystem`] 基于 [`HfApi`] 进行构建，并提供了常见的文件系统操作接口，如 `cp`、`mv`、`ls`、`du`、`glob`、`get_file` 和 `put_file`。

## 使用方式

请运行以下代码：

```python
>>> from huggingface_hub import HfFileSystem
>>> fs = HfFileSystem()

>>> # List all files in a directory
>>> fs.ls("datasets/my-username/my-dataset-repo/data", detail=False)
['datasets/my-username/my-dataset-repo/data/train.csv', 'datasets/my-username/my-dataset-repo/data/test.csv']

>>> # List all ".csv" files in a repo
>>> fs.glob("datasets/my-username/my-dataset-repo/**.csv")
['datasets/my-username/my-dataset-repo/data/train.csv', 'datasets/my-username/my-dataset-repo/data/test.csv']

>>> # Read a remote file 
>>> with fs.open("datasets/my-username/my-dataset-repo/data/train.csv", "r") as f:
...     train_data = f.readlines()

>>> # Read the content of a remote file as a string
>>> train_data = fs.read_text("datasets/my-username/my-dataset-repo/data/train.csv", revision="dev")

>>> # Write a remote file
>>> with fs.open("datasets/my-username/my-dataset-repo/data/validation.csv", "w") as f:
...     f.write("text,label")
...     f.write("Fantastic movie!,good")
```

您可以通过可选的 revision 参数来执行来自特定提交（例如分支、标签名称或提交哈希）的操作。

与Python内置的`open`不同的是，`fsspec`的`open`默认为二进制模式，即`"rb"`。这意味着您必须显式将模式设置为`"r"`以进行文本模式的读取，以及`"w"`以进行写入。目前还不支持追加到文件（模式为`"a"`和`"ab"`）。

## 集成

[`HfFileSystem`] 可以与任何集成了 `fsspec` 的库一起使用，只要 URL 遵循以下格式：

```
hf://[<repo_type_prefix>]<repo_id>[@<revision>]/<path/in/repo>
```

`repo_type_prefix` 对于数据集是 `datasets/`，对于工作空间是 `spaces/`，而模型在 URL 中不需要前缀。

以下列举了一些有趣的集成，其中使用[`HfFileSystem`] 的接口的所举的例子简化了与 Hub 互动的过程：

* 从/向 Hub 存储库读取/写入 [Pandas](https://pandas.pydata.org/pandas-docs/stable/user_guide/io.html#reading-writing-remote-files) DataFrame

请运行以下代码：

  ```python
  >>> import pandas as pd

  >>> # Read a remote CSV file into a dataframe
  >>> df = pd.read_csv("hf://datasets/my-username/my-dataset-repo/train.csv")

  >>> # Write a dataframe to a remote CSV file
  >>> df.to_csv("hf://datasets/my-username/my-dataset-repo/test.csv")
  ```

相同的工作流程也可以用于 [Dask](https://docs.dask.org/en/stable/how-to/connect-to-remote-data.html) 和 [Polars](https://pola-rs.github.io/polars/py-polars/html/reference/io.html) DataFrame。

* 使用 [DuckDB](https://duckdb.org/docs/guides/python/filesystems) 查询（远程）Hub 文件

请运行以下代码：

  ```python
  >>> from huggingface_hub import HfFileSystem
  >>> import duckdb

  >>> fs = HfFileSystem()
  >>> duckdb.register_filesystem(fs)
  >>> # Query a remote file and get the result back as a dataframe
  >>> fs_query_file = "hf://datasets/my-username/my-dataset-repo/data_dir/data.parquet"
  >>> df = duckdb.query(f"SELECT * FROM '{fs_query_file}' LIMIT 10").df()
  ```

* 使用 Hub 作为 [Zarr](https://zarr.readthedocs.io/en/stable/tutorial.html#io-with-fsspec) 的数组存储

请运行以下代码：

  ```python
  >>> import numpy as np
  >>> import zarr

  >>> embeddings = np.random.randn(50000, 1000).astype("float32")

  >>> # Write an array to a repo
  >>> with zarr.open_group("hf://my-username/my-model-repo/array-store", mode="w") as root:
  ...    foo = root.create_group("embeddings")
  ...    foobar = foo.zeros('experiment_0', shape=(50000, 1000), chunks=(10000, 1000), dtype='f4')
  ...    foobar[:] = embeddings

  >>> # Read an array from a repo
  >>> with zarr.open_group("hf://my-username/my-model-repo/array-store", mode="r") as root:
  ...    first_row = root["embeddings/experiment_0"][0]
  ```

## 身份验证

在许多情况下，您必须使用 Hugging Face 帐户进行登录以便与 Hub 进行交互。请参阅文档中的 [登录](../quick-start#login) 部分，了解有关 Hub 上身份验证方法的更多信息。

您也可以通过将您的 `token` 令牌作为参数传递给 [`HfFileSystem`] ，以便来以编程方式进行登录：

请运行以下代码：

```python
>>> from huggingface_hub import HfFileSystem
>>> fs = HfFileSystem(token=token)
```

注意：如果以这种方式登录，请小心在分享源代码时不要意外泄漏令牌！
