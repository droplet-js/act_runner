# act_runner

> 不推荐 Docker 模式: 个人认为与 drone_runner 相比，act_runner 在 Docker 这块的实现相对简单，而且还需要额外配置

### Gitea Action Runner 寄生到 Github Action

> 寄生时注意隐藏日志

* Linux Runner: 默认模式即可

```yaml
name: Gitea Runner on Github Action

on: 
  workflow_dispatch:

jobs:
  runner-exec:
    name: Exec Runner on ${{ matrix.os }}
    timeout-minutes: 360
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache/restore@v4
        with:
          path: |
            ./.runner
          key: runner-exec-${{ runner.os }}
      - name: Run gitea/act_runner-linux-amd64
        timeout-minutes: 180
        continue-on-error: true
        run: |
          curl -L https://gitea.com/gitea/act_runner/releases/download/v0.2.10/act_runner-0.2.10-linux-amd64 -o act_runner
          chmod a+x ./act_runner
          if [ ! -e ./.runner ]; then
            ./act_runner register --no-interactive --instance ${{ secrets.GITEA_INSTANCE_URL }} --token ${{ secrets.GITEA_RUNNER_REGISTRATION_TOKEN }} --name github_exec
          fi
          ./act_runner daemon  > /dev/null 2>&1
      - uses: actions/cache/save@v4
        with:
          path: |
            ./.runner
          key: runner-exec-${{ runner.os }}
```

```yaml
name: Gitea Action

on:
  push:
    paths-ignore:
      - '.gitea/workflows/**'
      - 'README.md'
    branches:
      - master
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ gitea.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by Gitea!"
      - run: echo "🔎 The name of your branch is ${{ gitea.ref }} and your repository is ${{ gitea.repository }}."
      - name: Check out repository code
        uses: actions/checkout@v4
      - run: echo "💡 The ${{ gitea.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ gitea.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."
```

* Macos Runner，不能用默认配置

```yaml
name: Gitea Runner on Github Action

on: 
  workflow_dispatch:

jobs:
  runner-exec-host:
    name: Exec Runner Host on ${{ matrix.os }}
    timeout-minutes: 360
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [macos-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache/restore@v4
        with:
          path: |
            ./.runner
          key: runner-exec-host-${{ runner.os }}
      - name: Run gitea/act_runner-darwin-amd64
        timeout-minutes: 180
        continue-on-error: true
        run: |
          curl -L https://gitea.com/gitea/act_runner/releases/download/v0.2.10/act_runner-0.2.10-darwin-amd64 -o act_runner
          chmod a+x ./act_runner
          if [ ! -e ./.runner ]; then
            ./act_runner register --no-interactive --instance ${{ secrets.GITEA_INSTANCE_URL }} --token ${{ secrets.GITEA_RUNNER_REGISTRATION_TOKEN }} --name github_macos_host --labels macos:host
          fi
          ./act_runner daemon  > /dev/null 2>&1
      - uses: actions/cache/save@v4
        with:
          path: |
            ./.runner
          key: runner-exec-host-${{ runner.os }}
```

```yaml
name: Gitea Action

on:
  push:
    paths-ignore:
      - '.gitea/workflows/**'
      - 'README.md'
    branches:
      - master
      - main

jobs:
  build:
    runs-on: macos # act_runner --labels macos:host
    steps:
```

* Windows Runner，不能用默认配置

> 暂时还不知道如何关闭日志

```yaml
name: Gitea Runner on Github Action

on: 
  workflow_dispatch:

jobs:
  runner-exec-host:
    name: Exec Runner Host on ${{ matrix.os }}
    timeout-minutes: 360
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [windows-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache/restore@v4
        with:
          path: |
            ./.runner
          key: runner-exec-host-${{ runner.os }}
      - name: Run gitea/act_runner-windows-amd64
        timeout-minutes: 180
        continue-on-error: true
        run: |
          curl -L https://gitea.com/gitea/act_runner/releases/download/v0.2.10/act_runner-0.2.10-windows-amd64.exe -o act_runner.exe
          if (-not (Test-Path .\.runner)) {
            .\act_runner.exe register --no-interactive --instance ${{ secrets.GITEA_INSTANCE_URL }} --token ${{ secrets.GITEA_RUNNER_REGISTRATION_TOKEN }} --name github_windows_host --labels windows:host
          }
          .\act_runner.exe daemon 
      - uses: actions/cache/save@v4
        with:
          path: |
            ./.runner
          key: runner-exec-host-${{ runner.os }}
```

```yaml
name: Gitea Action

on:
  push:
    paths-ignore:
      - '.gitea/workflows/**'
      - 'README.md'
    branches:
      - master
      - main

jobs:
  build:
    runs-on: windows # act_runner --labels windows:host
    steps:
```
