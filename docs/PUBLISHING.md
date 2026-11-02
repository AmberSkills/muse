# 把 MUSE 带到 GitHub

介绍文档放在仓库首页，安装包作为 Release 附件发布。用户打开仓库就能认识 MUSE，再去 Releases 下载。

## 1. 准备上传文件

在完整源码工程根目录运行：

```sh
npm run release:prepare
```

这个命令整理已有安装包，不重新构建，也不会自动上传任何文件。
如果还没有 DMG，先在 Apple Silicon Mac 上运行 `npm run desktop:build`。

输出目录如下：

```text
github-upload/
├── repository/                 # 把里面的内容上传到仓库根目录
│   ├── README.md
│   └── docs/
│       ├── DEVELOPMENT.md
│       ├── PUBLISHING.md
│       ├── RELEASE_NOTES.md
│       └── images/
│           ├── app-icon.png
│           ├── stage-dark.jpg
│           ├── stage-light.jpg
│           ├── mini-dancer.gif
│           └── README.md
└── release/                    # 安装包与校验文件放到 Release
    ├── MUSE_1.0.0_aarch64.dmg
    ├── SHA256SUMS.txt
    └── RELEASE_NOTES.md
```

这是用于发布介绍与安装包的最小文件集合，不包含源代码。

## 2. 上传首页文档

在 GitHub 创建你自己的仓库，把 `repository/` **里面的** `README.md` 和整个 `docs/` 目录上传到仓库根目录。
保留相对路径，这样 README 中的图标和文档链接才会正常显示。
不要把 `repository/` 这一层目录也放进仓库根目录。

上传后看一眼首页：应当能看到跳舞图标、深色舞台大图、浅色舞台和迷你少女动图。
图片路径都是相对路径，上传时不要遗漏 `docs/images/`。

## 3. 发布安装包

进入仓库的 **Releases**，新建一个 Release：

- **Tag**：当前版本使用 `v1.0.0`。如果该标签已发布，应为下一版递增版本号。
- **Title**：`MUSE 1.0.0 · 桌面跳舞少女`
- **Description**：复制 `release/RELEASE_NOTES.md` 的内容。
- **Assets**：上传 `MUSE_1.0.0_aarch64.dmg` 和 `SHA256SUMS.txt`。

先保存草稿，检查附件上传完成、说明无误后再发布。
GitHub 自动生成的 Source code 压缩包是仓库内容，不是可安装的 App；用户应下载 DMG。

官方操作说明：

```text
https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository
```

## 4. 核对文件

在本地上传目录中验证 SHA-256：

```sh
cd github-upload/release
shasum -a 256 -c SHA256SUMS.txt
```

输出 `MUSE_1.0.0_aarch64.dmg: OK` 表示文件与随附校验值一致。
这用于检查完整性，不能替代发布者身份签名或 Apple 公证。

## 下次更新

修改 App 后重新构建安装包，更新版本号、README 的下载文件名和版本说明，再运行 `npm run release:prepare`。
整理脚本读取 Tauri 配置中的版本号，文档版本未同步时会停止，避免混用不同版本。

`github-upload/` 是生成目录，已加入 `.gitignore`。整理脚本只处理明确列出的文件，不会上传项目中的其他内容。
不要将 `node_modules/`、`src-tauri/target/` 或本地配置文件整体拖进仓库。

如果以后还想发布完整源码，需要同时处理原始 GLB 资源的分发方式；当前源码构建流程会读取这些资源。
项目目前没有声明开源许可证，本文档也不代表对模型、动作或图像素材授予再分发许可。
