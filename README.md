# 狸群第二届全代会 · 网上投票

**线上地址：https://ClaydenPyridine.github.io/lq-vote/**（仓库 ClaydenPyridine/lq-vote，GitHub Pages，main 分支根目录）

## 部署说明

## 一、这是什么

一个纯静态的投票网页。选民打开链接，输入 **自己的 QQ 号 + 一次性投票码**，验证通过后填写四张选票、提交。

页面里**看不到任何 QQ 号**：`roster.js` 里只有每个「QQ号＋投票码」的哈希值，源码被拿走也反推不出名单（每人的码是 6 位、哈希迭代 200 轮）。

**选票结构**：26 届 4 选 1（差额）；27 届 3 选 2（差额，最多勾两人）；28 届 2 人逐人表态（等额）；教练 1 人（等额）。共 5 个选择项。

**专属链接**：网址后加 `?k=投票码`（如 `https://xxx/?k=LQ-2ND-UG8SKS`），打开后投票码自动填入，投票人只需填 QQ 号。`make_roster.py` 的第三个参数可自动生成整张链接表。

## 二、文件

| 文件 | 作用 |
|---|---|
| `index.html` | 投票页（唯一入口） |
| `roster.js` | 选民册哈希表，由 `make_roster.py` 生成，**不要手改** |
| `roster_投票码.csv` | 每人一行「QQ号、姓名、投票码」，**私下保管，别发群** |

## 三、部署（三选一，都不花钱）

### 1. GitHub Pages（最省事）
1. 新建一个仓库（Public），把 `index.html`、`roster.js` 传上去。
2. Settings → Pages → Source 选 `main` 分支、根目录，保存。
3. 一两分钟后得到地址：`https://<用户名>.github.io/<仓库名>/`，把它发群里即可。

### 2. Cloudflare Pages
1. 登录 Cloudflare → Workers & Pages → Create → Pages → Upload assets。
2. 把两个文件拖进去，部署，得到 `https://xxx.pages.dev`。

### 3. 任意静态托管
阿里云 OSS、腾讯云 COS、Netlify、Vercel 都行，只要能把两个文件放到同一个目录。

> 不建议用 `file://` 双击打开：手机上不好传，也没有固定链接。

## 四、更新名单

1. 把 QQ 号名单整理成 CSV（一行一人）：

   ```
   qq,name
   123456789,苏27 普狸策
   987654321,京28 雪碧猫咪
   ```

2. 运行：

   ```
   python make_roster.py 名单.csv web/roster.js
   ```

   会同时生成 `roster_投票码.csv`（QQ号、姓名、投票码），按行私聊发给本人。

3. 把新的 `roster.js` 覆盖上传，刷新即生效。

## 五、上线前必改

打开 `index.html`，找到 `const CONFIG = {...}`：

- `TEST_MODE: true` → 改成 `false`（否则不校验投票时间，随时可投）
- `OPEN_AT` / `CLOSE_AT`：投票时间窗口，默认 11-10 09:00 至 11-12 00:00
- `REPORT_ENDPOINT`：留空则只生成回执；填了就把选票 POST 过去（Supabase、Cloudflare Worker、Formspree 都能收）

## 六、收票的两种方式

**A. 回执（默认，零配置）**
选民提交后点「下载/保存选票回执」，把 `.json` 或复制的文本发给主席团；主席团丢进同一个文件夹，用 `计票台.html` 导入汇总，或直接把文件夹交给助手读取。

**B. 自动上报（需配置）**
在 `REPORT_ENDPOINT` 填一个接收地址（例如 Cloudflare Worker + KV，或 Supabase 的 REST 接口），选票会随提交同步过去。

## 七、防重复

- 一码一人：哈希表里没有「QQ号+码」的组合，进不去。
- 本机记忆：同一浏览器投过就拦住（localStorage）。
- 汇总去重：计票台按「回执编号 / QQ号 / 投票码」三重去重，换浏览器重投也会被识别。
