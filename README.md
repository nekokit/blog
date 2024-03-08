# Neko Blog

这是一个使用了 [Hugo theme Stack](https://github.com/CaiJimmy/hugo-theme-stack) 的博客。

## 环境迁移

1. `git clone https://github.com/nekokit/blog.git`
2. `code blog`
3. 使用 devcontainer 打开

## 手动更新主题

如果主题的 `v4` 版本发布，则需要手动更新主题（修改 `config/module.toml` ）。

```shell
hugo mod get -u github.com/CaiJimmy/hugo-theme-stack/v3
hugo mod tidy
```
