# Blind Box Weight Database 维护交接

这是盲盒重量与摇盒手感数据库。线上页面：

https://blindbox-weight.islandsapp.workers.dev/

## 给接手同事和 AI 的一句话说明

只手工编辑结构化数据 `data.json`，同时保持本机两份数据一致；`index.html` 必须由脚本生成，不能手工修改其中的压缩数据。更新后检查差异、提交公开仓库，并用带随机参数的线上地址确认新内容已经生效。

## 本机目录

| 用途 | 路径 |
| --- | --- |
| 数据源及同步脚本 | `D:\Claude\data-assistant\blindbox-db` |
| 公开仓库及线上页面 | `D:\Claude\data-assistant\blindbox-weight` |

需要同步维护：

- `blindbox-db/data.json`
- `blindbox-weight/data.json`

公开仓库正常只提交：

- `data.json`
- `index.html`
- 文档类文件，例如本 README

`blindbox-db` 中可能存在其他人的本地修改、数据库文件或未跟踪文件。不要清理、回退或提交与当前任务无关的内容。

目前 `blindbox-db` 是本机数据源工作目录，默认不提交；`blindbox-weight` 是需要提交和推送的公开仓库。只有用户另行指定时才提交 `blindbox-db`。

## 数据结构

```json
{
  "series": [
    {
      "id": "稳定且唯一的英文短横线 ID",
      "name": "IP|Official English Series Name",
      "brand": "POP MART",
      "ip": "IP 名称（部分旧数据可能没有）",
      "releaseDate": "YYYY-MM-DD",
      "source_url": "资料来源（如有）",
      "note": "整套系列备注（如有）",
      "figures": [
        {
          "name": "Official English Figure Name",
          "weight": 123.4,
          "feel": "English hand-feel description",
          "spuId": "SPU ID",
          "image": "https://..."
        }
      ]
    }
  ]
}
```

注意：旧数据中 `weight` 可能是数字，也可能是带 `g`、`+` 或范围的字符串。除非用户明确要求整理格式，否则保留原值。

## 数据录入规则

1. 用户给出的重量和手感按原意保存，不自行推测、补全或改写数值。
2. 页面面向英文用户：系列名、款式名、备注和手感使用英文。用户提供中文手感时应忠实翻译成自然英文，不增加原文没有的信息。
3. 官方英文名优先采用品牌官网或官方商品图；找不到可靠官方译名时先保留，并列给用户确认。
4. 隐藏款统一在名称后标记 ` (Secret)`；超级隐藏可使用 ` (Super Secret)`。
5. 只有款式、但没有重量或手感资料时，不要编造。用户要求只统计“手感为空”时，有重量但没有手感的款式不计入。
6. 修改系列名称时，不要顺带改动 ID、款式、重量、手感、SPU 或图片。
7. 新增数据前检查系列 `id` 是否唯一；同一系列内检查款式和 SPU 是否重复。
8. 评论区及用户可见文案使用英文，避免残留中文。

## 标准更新流程

以下命令在 PowerShell 中执行。

### 1. 开始前检查工作区

```powershell
git -C D:\Claude\data-assistant\blindbox-db status --short
git -C D:\Claude\data-assistant\blindbox-weight status --short
git -C D:\Claude\data-assistant\blindbox-weight remote -v
git -C D:\Claude\data-assistant\blindbox-weight branch --show-current
```

公开仓库应为预期的 `origin`，部署分支必须是 `main`。如果当前不是 `main`，先停止并确认原因，不要直接执行后面的推送命令。如目标文件已经有其他修改，先阅读差异并保留；如果与本次目标重叠且无法安全合并，应先向用户说明，不要覆盖。

### 2. 更新两份 JSON

对 `blindbox-db/data.json` 和 `blindbox-weight/data.json` 做相同修改。优先使用小范围补丁，不要整文件格式化。

### 3. 生成两个页面

```powershell
cd D:\Claude\data-assistant\blindbox-db
node sync_to_html.js

cd D:\Claude\data-assistant\blindbox-weight
node ..\blindbox-db\sync_to_html.js
```

脚本成功时会输出 `Synced data.json to index.html`。脚本依赖当前目录，因此不要省略上面的 `cd`。

### 4. 检查 JSON、一致性和差异

```powershell
cd D:\Claude\data-assistant
node -e "const fs=require('fs');const a=JSON.parse(fs.readFileSync('blindbox-db/data.json','utf8'));const b=JSON.parse(fs.readFileSync('blindbox-weight/data.json','utf8'));console.log('same:',JSON.stringify(a)===JSON.stringify(b))"

git -C blindbox-weight diff --check
git -C blindbox-weight diff --stat
git -C blindbox-weight diff -- data.json
git -C blindbox-weight diff -- index.html
```

必须确认：

- JSON 可以正常解析。
- `same: true`，表示两份 JSON 的内容及字段顺序完全一致。
- 差异只包含本次要求的内容。
- 重量、手感、SPU 和图片没有被意外覆盖。
- `index.html` 的变化来自同步脚本，不包含其他页面改动。

### 5. 提交并推送公开仓库

```powershell
cd D:\Claude\data-assistant\blindbox-weight
git add -- data.json index.html
git commit -m "简要说明本次更新"
git push origin main
```

如本次修改了 README，再将 `README.md` 加入 `git add`。不要使用 `git add .`。

### 6. 验证线上版本

推送后访问带缓存参数的地址：

```text
https://blindbox-weight.islandsapp.workers.dev/?verify=本次提交短哈希
```

当前发布方式是推送 `origin/main` 后由线上环境自动更新。不要只看 Git 推送成功：需要按系列名或款式名定位，确认线上页面显示本次修改的字段；如果是替换操作，还要确认旧内容已经消失。

Cloudflare 更新偶尔需要等待片刻。如果尚未生效，等待后使用新的 `verify` 参数重新检查；不要为了触发部署重复提交相同内容。长时间仍未更新时，检查 GitHub 部署状态或联系项目维护者，不能把未验证的版本报告为已上线。

## 常见任务

### 新增系列

- 先收集系列官方英文名、款式、SPU、图片、重量和手感。
- 没有可用资料的款式可以暂不录入，并在系列 `note` 中说明。
- 新系列放入 `series` 数组，不要破坏周围 JSON 逗号和括号。

### 补充重量或手感

- 用系列 ID、款式名或 SPU 精确定位。
- 只改目标字段。
- 重量单位默认是克；保留用户给出的 `+`、范围及特殊说明。

### 修正英文名称

- 优先核对官方品牌页面或官方商品图。
- 官方看似不自然的英文也应保留，不要因为像机翻就自行润色。
- 无法确认的名称先列给用户，不要直接替换。

### 删除系列或款式

- 必须有用户明确指令。
- 同时从两份 `data.json` 删除，再重新生成两个 `index.html`。

## 禁止事项

- 不要直接编辑 `index.html` 中的内嵌数据。
- 不要只更新一份 `data.json`。
- 不要猜测重量、手感、SPU、隐藏款或官方译名。
- 不要覆盖或清理不属于本次任务的本地修改。
- 不要在未经要求时提交评论系统、后台页面或 Workers 配置。
- 不要把账号、Cookie、Token 或其他凭证写入仓库、聊天或文档。

## AI 接手时建议先做

1. 阅读本 README。
2. 查看两边的 `git status`，保护已有修改。
3. 搜索目标系列或 SPU，确认将要修改的唯一记录。
4. 用最小补丁同步修改两份 `data.json`。
5. 按“标准更新流程”完成生成、校验、提交和线上验证。

