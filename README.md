# Hexo-site

**说明**：此仓库为个人博客源码目录

# 项目已经迁移到 [whg517/whg517.github.io](https://github.com/whg517/whg517.github.io)

## 使用说明

博客使用 [Hexo 博客框架](https://hexo.io/zh-cn/docs/) 详细操作请参考官方文档。

主题使用 [«NexT» Theme](https://github.com/theme-next/hexo-theme-next)。具体使用参考文档。

1. 安装 hexo 环境。参考 [Hexo 概述](https://hexo.io/zh-cn/docs/)

```bash
npm install -g hexo-cli
```

2. 进入源码 `blog` 目录安装依赖

```bash
yarn install
```

3. 获取主题。参考 [下载 tag 指向的 release 版本](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/INSTALLATION.md#%E9%80%89%E9%A1%B9-2%E4%B8%8B%E8%BD%BD-tag-%E6%8C%87%E5%90%91%E7%9A%84-release-%E7%89%88%E6%9C%AC)

```bash
git clone --branch v7.8.0 https://github.com/theme-next/hexo-theme-next themes/next
```

4. [生成静态文件](https://hexo.io/zh-cn/docs/commands#generate)

```bash
hexo generate
```

5. [预览](https://hexo.io/zh-cn/docs/commands#server)

```bash
hexo server
```

6. [部署到 Github](https://hexo.io/zh-cn/docs/commands#deploy)

```bash
hexo deploy
hexo -g deploy # 部署前生成静态资源
```

## 配置说明

### Hexo 配置

参考 [Hexo 配置](https://hexo.io/zh-cn/docs/configuration) 文档

### 主题配置

由于主题使用的 NexT，所以仅说明 NexT 主题配置。如果以后使用了其他主题，再增加说明

NexT 的主题配置默认情况下是放在主题目录下的 `_config.yml` 的，但是由于获取新主题代码，可能存在冲突等诸多使用不便利问题，现使用 NexT 
提供的 [数据文件配置](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/DATA-FILES.md#%E9%80%89%E6%8B%A9-1hexo-%E6%96%B9%E5%BC%8F) 
方式。即在 Hexo 的配置文件中增加 `theme_config:` 配置项，主题配置均在这个配置下面。

## 变更记录

[变更记录](./docs/CHANGELOG.md)