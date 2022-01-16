# nvm

> nvm全名node.js version management，顾名思义是一个nodejs的版本管理工具。通过它可以安装和切换不同版本的nodejs

## 安装

[github官方文档](https://github.com/nvm-sh/nvm)

```sh
# 安装版本0.33.8, 最新版本0.39.0
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# 测试是否安装成功
nvm --version
# 若提示：command not found: nvm，继续以下步骤

# 若安装了oh my zsh，以下配置会自动写入~/.zshrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

# 输入上方的配置
sudo vim ~/.bash_profile

# 启用配置
source ~/.zshrc
```

### M1 芯片问题

macOS m1 对nvm有兼容性问题处理

安装低版本的node时，报错：error:** **Target architecture arm64 is only supported on arm64 and x64 host**

1. 升级nvm到最新版本（用上诉方法，去官网查看最新版本，并进行升级）
2. 执行一下命令

```sh
# 查看node当前版本类型
node -p process.arch
# 若是arm64, 则执行一下操作 ?? 好像有问题 =。= 太难了
softwareupdate --install-rosetta
arch -x86_64 brew install zsh
```

## 使用

```sh
nvm install [node.js版本号]
```



