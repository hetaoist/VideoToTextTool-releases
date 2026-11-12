# VideoToTextTool Releases

本仓库**只存放发布产物与更新清单**，不存放源码。源码与问题反馈请前往主仓库：
<https://github.com/hetaoist/VideoToTextTool>（当前私有，后期开源）。

面向 Windows x64。应用内「帮助 → 检查更新」会读取本仓库的最新 Release 并完成下载、校验与安装。

## 手动下载与安装

1. 打开 [Releases](https://github.com/hetaoist/VideoToTextTool-releases/releases)，选择最新版本；
2. 下载 `VideoToTextTool-<版本>-win64.zip`；
3. 先退出正在运行的 VideoToTextTool；
4. 将 zip 解压到**可写目录**（如 `D:\Apps\VideoToTextTool`），运行 `VideoToTextTool.exe`。

> 不建议解压到 `C:\Program Files`：应用内自动更新需要写入安装目录，权限不足时会被拒绝，
> 只能手动替换。便携使用请保持整个目录在一起，不要只拷贝 exe。

## 校验下载文件（SHA256）

每次发布都附带 `SHA256SUMS.txt`。在 PowerShell 中执行：

```powershell
Get-FileHash .\VideoToTextTool-1.0.0-win64.zip -Algorithm SHA256
```

把输出的哈希与 `SHA256SUMS.txt` 中对应文件的值逐字比对。不一致时不要运行，重新下载。

> 当前发布包**没有代码签名**。SHA256 只能证明文件完整、未被篡改，不能证明发布者身份；
> 请只从本仓库下载，并核对哈希。

## 自动更新怎么工作

1. 应用启动后（若开启）静默检查，或在「帮助 → 检查更新」手动检查；
2. 读取最新 Release 的 `latest.json`，按语义化版本比较，只接受更高版本；
3. 下载全量 zip，支持断点续传与取消；
4. 下载完成后校验大小与 SHA256，不符立即删除并报错；
5. 用户确认后退出应用，由同目录的 `VideoToTextToolUpdater.exe` 等待主进程退出，
   逐文件替换并备份；
6. 任一步失败自动回滚到旧版本，应用仍可启动；成功后自动重启。

安装目录必须可写，且同目录下存在 `VideoToTextToolUpdater.exe`。更新失败不会影响当前版本使用。

## 资产说明

| 文件 | 作用 |
| --- | --- |
| `VideoToTextTool-<版本>-win64.zip` | 全量安装包，zip 根目录即程序文件 |
| `latest.json` | 更新清单：版本、说明、发布时间、预发布标记、`payload_id`、全量包名称/大小/SHA256、需删除的旧文件 |
| `SHA256SUMS.txt` | 本次发布所有资产的 SHA256 |
| `RELEASE_NOTES.md` | 与 Release 页面正文一致的发行说明 |

`latest.json` 关键字段：

```json
{
  "schema": 1,
  "version": "1.0.0",
  "notes": "## 变更分类 ...",
  "published_at": "2026-11-12T10:00:00Z",
  "prerelease": false,
  "payload_id": "py312-req...-ffmpeg...",
  "assets_base": "https://github.com/hetaoist/VideoToTextTool-releases/releases/download/v1.0.0/",
  "current": {
    "name": "VideoToTextTool-1.0.0-win64.zip",
    "size": 424000000,
    "sha256": "..."
  },
  "delete": []
}
```

## 版本号与 tag 约定

- tag 使用 `v<版本>`，例如 `v1.0.0`、`v1.1.0-rc.1`；
- `latest.json` 的 `version` 不带 `v`，必须与 `core/version.py` 一致；
- 预发布版本需在 GitHub 上标记为 Pre-release，并在清单中写 `"prerelease": true`；
  客户端默认不接收预发布；
- 不发布降级包：客户端只接受比当前版本更高的版本。

## 自动更新失败怎么排查

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| 「发布仓库未公开或不存在」（404） | 仓库未公开、没有 Release，或客户端配置的仓库不对 | 确认仓库为 Public 且已有 Release；检查设置 → 更新 → 发布仓库 |
| 「请求过于频繁，请稍后重试」 | GitHub API 匿名限流 | 稍后重试；频繁检查时可配置静态清单或代理 |
| 「连接更新服务超时」 | 网络/代理问题 | 检查网络；在设置中配置代理或镜像 |
| 「更新信息格式错误」 | `latest.json` 非法 JSON 或版本号不合规 | 重新发布正确的清单 |
| 下载很慢或中断 | 国际链路 | 在设置 → 更新中配置下载镜像或代理；中断后重试可续传 |
| 无法一键安装 | 安装目录不可写 / 缺少 `VideoToTextToolUpdater.exe` | 移到可写目录，或手动下载全量包替换 |
| 更新后启动失败 | 替换异常 | 备份目录在安装目录同级，名为 `.vtt-update-backup-<版本>-<pid>`；用其中文件手工恢复，并在主仓库反馈 |

设置路径：应用内「文件 → 设置 → 更新」。
