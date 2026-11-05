# 第三方资源镜像（不随脚本发布）

本目录存放**不便直接依赖上游地址**的第三方静态资源镜像，作为 `chaoxing-assistant-clean` 发布仓库的备用托管源。

- 本目录**不进入** `chaoxing-assistant-clean/` 发布仓库，也不随 Greasyfork / 脚本猫 的脚本包分发。
- 目录结构刻意保持与上游 URL 路径一致，便于直接平替。
- 目的：上游域名一旦失效或审核存疑，把脚本头里的 `@resource` 换成这里的 GitHub Raw / jsDelivr 地址即可，**无需改动任何逻辑代码**。

## 目录

```
第三方资源镜像/
├── README.md                 本文件
├── MANIFEST.json             资源清单：来源、许可、大小与 SHA-256
├── LICENSE-MIT              上游 mit 许可与版权声明
└── ttf/2.0/table.json       学习通 font-cxsecret 字体映射表（355,427 字节）
```

## 资源一览

| 文件 | 大小 | SHA-256 |
| --- | --- | --- |
| `ttf/2.0/table.json` | 355,427 字节 | `FBC6AE3C795A46CA2BD0994E68E7CB90CA4D9A046F335512657760D9A4FDF875` |

## table.json 是什么

学习通部分题库页面会把题干渲染成 `font-cxsecret` 字体加密文本：每个汉字对应一个私有字形，肉眼正常、复制出来是乱码。

`table.json` 是「字形指纹 → 真实汉字」的字典：

- 键：该字符字形路径 JSON 的 MD5 值后 8 位（长度 6–9 位不等）。
- 值：真实字符的 Unicode 码点，范围 `19968..40869`（U+4E00..U+9FA5）。
- 条目数：20,906 条（其中 20,902 条为映射，另有 4 条是原表作者写入的元数据，见下）。

原表自带元数据，可直接用于溯源：

```json
{ "author": "wyn665817", "email": "wyn665817@163.com", "license": "MIT", "version": "20220515.1" }
```

脚本侧用法（`超星学习通助手.user.js` 中的 `removeF()`）：解析页面内联的 base64 字体文件 → 用 Typr.js 遍历 U+4E00–U+9FA5 取每个字的字形路径 → `MD5(...).slice(24)` → 查表替换 DOM 文本。

## 来源与许可

| 项 | 内容 |
| --- | --- |
| 原始出处 | 脚本猫 432《超星字体解密》作者 **wyn665817**（`wyn665817@163.com`） |
| 许可 | MIT（详见 [LICENSE-MIT](./LICENSE-MIT)） |
| 表内声明版本 | `20220515.1` |
| 此前脚本使用的地址 | `https://www.forestpolice.org/ttf/2.0/table.json`（HTTP 200，非官方托管） |
| 本镜像是 | **原样逐字节复制**，未做任何修改、裁剪或重新生成 |

`forestpolice.org` 只是该表的一个静态托管点，**与原作者身份、授权关系无关**。此处保留副本是为了不把唯一可用性押在一个第三方个人域名上。

## 如何换成镜像地址

镜像推到 GitHub 后，把脚本头的这一行：

```js
// @resource      Table https://www.forestpolice.org/ttf/2.0/table.json
```

换成（二选一）：

```js
// @resource      Table https://cdn.jsdelivr.net/gh/<用户名>/<仓库名>@<tag>/ttf/2.0/table.json
// @resource      Table https://raw.githubusercontent.com/<用户名>/<仓库名>/<tag>/ttf/2.0/table.json
```

注意：

1. 用 **tag 或 commit SHA 固定版本**，不要用 `main`——主分支一变，所有已安装用户的字体表就跟着变，且无法审计。
2. jsDelivr 需要仓库有 tag 才会缓存，首次可能要等几分钟生效；Raw 地址在国内部分网络会被墙，jsDelivr 更稳。
3. 改 `@resource` 后必须同步改 `任务引擎回归测试.js` 里 `@resource` 域名白名单（目前含 `www.forestpolice.org`），否则测试会红。
4. `table.json` 的 MIME 必须是 `application/json`（GitHub Raw / jsDelivr 都满足）；若自建托管返回 `text/plain`，`GM_getResourceText` 仍能拿到字符串，不影响解析。

## 校验副本完整性

```powershell
Get-FileHash ".\ttf\2.0\table.json" -Algorithm SHA256
```

结果应为 `FBC6AE3C795A46CA2BD0994E68E7CB90CA4D9A046F335512657760D9A4FDF875`。

## 仓库地址（已上线）

- 镜像仓库：<https://github.com/hjm-al/chaoxing-font-table>
- 主仓库：<https://github.com/hjm-al/chaoxing-assistant>

主仓库脚本头部的 `@resource Table` 仍指向 `https://www.forestpolice.org/ttf/2.0/table.json`（上游可用时优先用它）。若该站失效，把那一行改成镜像仓库里的原始地址即可：

```
https://raw.githubusercontent.com/hjm-al/chaoxing-font-table/main/ttf/2.0/table.json
```

## 推送到 GitHub 的建议步骤

已执行完毕，留作日后重建备忘：

```powershell
cd "C:\Users\o\Desktop\脚本\第三方资源镜像"
git init
git add .
git commit -m "feat: mirror chaoxing font table (wyn665817, MIT)"
git branch -M main
git remote add origin git@github.com:hjm-al/chaoxing-font-table.git
git push -u origin main
git tag v1.0.0
git push origin v1.0.0
```

仓库 `README.md` 里请保留上文「来源与许可」整节，MIT 要求保留版权声明与许可全文。
