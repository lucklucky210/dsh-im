# FORK 说明

本仓库是 [xmanrui/dsh-im](https://github.com/xmanrui/dsh-im) 的个人 fork，用于维护上游暂时未处理的少量修复。
基线版本：v5.7.0（上游 commit e278ac7）。

## 与上游的差异

### 1. 图片消息不再伪造用户发言

上游行为：图片单独到达（没有文字）时，`src/channels/shared/image-prompt.mjs` 的
`promptContentForMessage()` 会把固定提示 `请分析这张图片。` 作为 `text` 段推进**用户消息体**，
于是 Host 记录下 `user/message` 的第一段就是这句用户从没打过的话，并立刻开始一个回合。

本 fork：改为非指令性的占位 `（图片消息：用户未附文字说明）`。只影响"消息里没有文字"这一种情况；
带 caption 的富文本消息、以及用户自己输入的文字完全不受影响。

上游关联：xmanrui/dsh-im#385（同一触发点，但上游把它当作 /hold 功能需求处理）。

## 构建

`lib/` 是提交进仓库的构建产物，运行时加载的是 `lib/index.js`，**只改 `src/` 不会生效**。

```bash
npm ci          # 需要 devDeps（esbuild、react 等）
npm run build   # plugin-src/client/build.mjs && plugin-src/host/build.mjs
```

注意：`lib/index.js` 里的中文以大写 `\uXXXX` 转义存放，直接 grep 原文搜不到；改文案要按转义串替换，
或改 `src/` 后重新构建。

## 本地安装 / 替换已装插件

```bash
cd ~/.dsh/profiles/web
dsh plugin --profile web add -w /绝对路径/到/本仓库
```

已通过 npm 安装过 `@xmanrui/dsh-im` 时，本地路径安装会覆盖它；回滚就重新
`dsh plugin --profile web add -w @xmanrui/dsh-im`。

## 同步上游

```bash
git remote add upstream https://github.com/xmanrui/dsh-im.git  # 仅首次
git fetch upstream main
git rebase upstream/main
```

上游如果自己修了图片补白，rebase 时会在 `image-prompt.mjs:7` 与 `lib/index.js` 的同一行冲突，按上游版本取值即可。

## 维护约定

- 每个修复一个分支、一个提交，提交信息写清"上游行为 / 本 fork 行为 / 上游 issue 编号"；
- 改了 `src/` 必须同时提交重新构建的 `lib/`，否则安装后无效果；
- 版本号不追上游发布节奏，跟随上游 tag 打本地 tag（如 `v5.7.0-fork.1`）。
