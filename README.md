# delete-workflow-runs v2
这个 GitHub Action 用于删除代码库中的 workflow 运行记录。这个 JavaScript 写成的 Action 包含了两个 Workflow Runs API：

* [**列出代码库的 workflows**](https://docs.github.com/cn/free-pro-team@latest/rest/reference/actions#list-repository-workflows) -- 列出代码库中所有可用的 workflows。

* [**列出 workflow 运行记录**](https://docs.github.com/cn/free-pro-team@latest/rest/reference/actions#list-workflow-runs) -- 列出指定 workflow 的所有运行记录。

* [**删除 workflow 运行记录**](https://docs.github.com/cn/free-pro-team@latest/rest/reference/actions#delete-a-workflow-run) -- 删除指定 workflow 运行记录。

该 Action 将计算每个 workflow 运行记录已保留的天数，然后使用此数字与您在输入参数 "[**`retain_days`**](#3-retain_days)" 中指定的数字进行比较。如果 workflow 运行记录的保留天数已达到（大于等于）指定的数字，则会删除该运行记录。

## 更新内容
* 添加输入参数 "[**`keep_minimum_runs`**](#4-keep_minimum_runs)". 通过这个参数，可以指定每个 workflow 的最小运行次数。即使某些运行记录已达到了指定的保留天数，也会保留最新的指定次数的运行记录。

* 优化代码以简化流程。
##

## 输入参数
### 1. `token`
#### 必须：是
#### 默认值：`${{ github.token }}`
用于身份验证的 token。
* 如果 workflow 运行记录在当前 repository 中，使用 **`github.token`** 就可以了。更多细节，请参阅 [**`GITHUB_TOKEN`**](https://docs.github.com/cn/free-pro-team@latest/actions/reference/authentication-in-a-workflow)。
* 如果 workflow 运行记录在其他 repository 中，则需要使用具有 **`repo`** 范围的 personal access token (PAT)。更多细节，请参阅 "[Creating a personal access token](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)"。

### 2. `repository`
#### 必须：是
#### 默认值：`${{ github.repository }}`
workflow 运行记录所在的代码库名称。

### 3. `retain_days`
#### 必须：是
#### 默认值：90
与每个 workflow 运行记录的保留天数进行比较的天数。

### 4. `keep_minimum_runs`
#### 必须：是
#### 默认值：6
每个 workflow 的最小运行次数。
##

## 示例
### 在 cron 定时任务中运行, 参见 [schedule event](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#schedule).
> **提示：** 推荐使用定时任务来自动删除旧的 workflow 运行记录。
```yaml
name: Delete old workflow runs
on:
  schedule:
    - cron: '0 0 1 * *'
# 每月 1 日 00:00 运行。

jobs:
  del_runs:
    runs-on: ubuntu-latest
    steps:
      - name: Delete workflow runs
        uses: ActionsRML/delete-workflow-runs@main
        with:
          token: ${{ secrets.AUTH_PAT }}
          repository: ${{ github.repository }}
          retain_days: 30
```

### 在手动触发的 workflow 中运行, 参见 [workflow_dispatch event](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows#workflow_dispatch).
> 您可以随时手动触发这个 workflow 来删除旧的 workflow 运行记录。 <br/>
![manual workflow](https://github.com/zhoujinshi/delete-workflow-runs/blob/main/img/example.PNG)
```yaml
name: Delete old workflow runs
on:
  workflow_dispatch:
    inputs:
      days:
        description: 'Number of days.'
        required: true
        default: 90

jobs:
  del_runs:
    runs-on: ubuntu-latest
    steps:
      - name: Delete workflow runs
