<!--⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.
-->

# 参与讨论和发起拉取请求

`huggingface_hub` 库提供了与 Hub 上的拉取请求和讨论互动的 Python 接口。
您可以访问[专门的文档页面](https://huggingface.co/docs/hub/repositories-pull-requests-discussions)以更深入地了解 Hub 上的讨论和拉取请求是什么，以及它们在底层是如何运作的。

## 从 Hub 获取讨论和拉取请求

`HfApi` 允许你检索给定仓库的讨论和拉取请求

请运行以下代码：

```python
>>> from huggingface_hub import get_repo_discussions
>>> for discussion in get_repo_discussions(repo_id="bigscience/bloom-1b3"):
...     print(f"{discussion.num} - {discussion.title}, pr: {discussion.is_pull_request}")

# 11 - Add Flax weights, pr: True
# 10 - Update README.md, pr: True
# 9 - Training languages in the model card, pr: True
# 8 - Update tokenizer_config.json, pr: True
# 7 - Slurm training script, pr: False
[...]
```

`HfApi.get_repo_discussions` 返回一个[生成器](https://docs.python.org/3.7/howto/functional.html#generators)，它生成 [`Discussion`] 对象。要获取所有讨论的单个列表，请运行：

```python
>>> from huggingface_hub import get_repo_discussions
>>> discussions_list = list(get_repo_discussions(repo_id="bert-base-uncased"))
```

[`HfApi.get_repo_discussions`] 返回的 [`Discussion`] 对象包含讨论或拉取请求的高级概览。你还可以使用 [`HfApi.get_discussion_details`] 获取更详细的信息：

请运行以下代码：

```python
>>> from huggingface_hub import get_discussion_details

>>> get_discussion_details(
...     repo_id="bigscience/bloom-1b3",
...     discussion_num=2
... )
DiscussionWithDetails(
    num=2,
    author='cakiki',
    title='Update VRAM memory for the V100s',
    status='open',
    is_pull_request=True,
    events=[
        DiscussionComment(type='comment', author='cakiki', ...),
        DiscussionCommit(type='commit', author='cakiki', summary='Update VRAM memory for the V100s', oid='1256f9d9a33fa8887e1c1bf0e09b4713da96773a', ...),
    ],
    conflicting_files=[],
    target_branch='refs/heads/main',
    merge_commit_oid=None,
    diff='diff --git a/README.md b/README.md\nindex a6ae3b9294edf8d0eda0d67c7780a10241242a7e..3a1814f212bc3f0d3cc8f74bdbd316de4ae7b9e3 100644\n--- a/README.md\n+++ b/README.md\n@@ -132,7 +132,7 [...]',
)
```

[`HfApi.get_discussion_details`] 返回一个 [`DiscussionWithDetails`] 对象，它是 [`Discussion`] 的子类，提供有关讨论或拉取请求的更详细信息。信息包括所有评论、状态更改以及通过 [`DiscussionWithDetails.events`] 进行的重命名。

对于拉取请求，你可以使用 [`DiscussionWithDetails.diff`] 检索原始的 git 差异。拉取请求的所有提交都列在 [`DiscussionWithDetails.events`] 中。


## 以编程方式创建和编辑讨论或拉取请求

[`HfApi`] 类还提供了创建和编辑讨论或拉取请求的方法。您需要一个[访问令牌](https://huggingface.co/docs/hub/security-tokens)来创建和编辑讨论或拉取请求。

通过[`create_commit`] API向Hub提出更改建议的最简单方法是：只需将`create_pr`参数设置为`True`。此参数还适用于其他封装了[`create_commit`]的方法：

- [`upload_file`]
- [`upload_folder`]
- [`delete_file`]
- [`delete_folder`]
- [`metadata_update`]

请运行以下代码：

```python
>>> from huggingface_hub import metadata_update

>>> metadata_update(
...     repo_id="username/repo_name",
...     metadata={"tags": ["computer-vision", "awesome-model"]},
...     create_pr=True,
... )
```

您还可以使用 [HfApi.create_discussion]（或 [HfApi.create_pull_request]）在仓库上创建讨论（或拉取请求）。
以这种方式打开拉取请求可能在您需要在本地进行更改时很有用。以这种方式打开的拉取请求将处于“draft（草稿）”模式。

请运行以下代码：

```python
>>> from huggingface_hub import create_discussion, create_pull_request

>>> create_discussion(
...     repo_id="username/repo-name",
...     title="Hi from the huggingface_hub library!",
...     token="<insert your access token here>",
... )
DiscussionWithDetails(...)

>>> create_pull_request(
...     repo_id="username/repo-name",
...     title="Hi from the huggingface_hub library!",
...     token="<insert your access token here>",
... )
DiscussionWithDetails(..., is_pull_request=True)
```

使用 [`HfApi`] 类完全可以进行拉取请求和讨论的管理。例如：

* [`comment_discussion`] 用于添加评论
* [`edit_discussion_comment`] 用于编辑评论
* [`rename_discussion`] 用于重命名讨论或拉取请求
* [`change_discussion_status`] 用于打开或关闭讨论/拉取请求
* [`merge_pull_request`] 用于合并拉取请求


访问[`HfApi`]文档页面，查看所有可用方法的详尽参考。

## 推送更改到拉取请求

*即将推出！*

## 浏览更多内容

为了获得更详细的参考资料，请访问[讨论和拉取请求](../package_reference/community)以及 [hf_api](../package_reference/hf_api) 文档页面。
