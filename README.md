# Drill to Earth's Core Wiki

Roblox《Drill to Earth's Core》的粉丝 wiki 与攻略站。

**域名**：https://drilltoearthscore.xyz（已购买，待接 Cloudflare）· **模板**：[AnvilWiki](https://github.com/PNGTRID/AnvilWiki)（MIT）

---

## 技术栈

| 层 | 选型 |
| --- | --- |
| 框架 | Astro 7（`output: 'static'`，零 JS 优先，Lighthouse 4×100） |
| 内容 | Content Collections + MDX，Zod 构建时硬校验（`src/content.config.ts`） |
| 样式 | Tailwind CSS 3 + CSS 变量主题（品牌色 `#ea580c`） |
| 部署 | Cloudflare Workers static assets，Git 集成自动构建 |
| 包管理 | pnpm 11（需 Node ≥ 22.13，仓库 `.nvmrc` = 22） |

## 目录结构（三层分离，改动前先读）

```
src/pages, src/components, src/lib   → 框架层（fork-once，不逐游戏改）
src/config, src/locales, globals.css → 配置层（游戏标识/主题/文案）
src/content/wiki/<locale>/<category> → 内容层（文章 MDX，随游戏更新）
```

详细规范见 [`AGENTS.md`](./AGENTS.md) 与 [`docs/content-format.md`](./docs/content-format.md)。

## 本地开发

```bash
pnpm install
pnpm dev            # http://localhost:4321

# 生产构建（SITE_URL 是构建时变量，必须带上）
SITE_URL=https://drilltoearthscore.xyz pnpm build
pnpm preview        # 预览 dist/
```

## 内容工作流

文章路径即 URL：`src/content/wiki/en/<category>/<slug>.mdx` → `/<category>/<slug>/`

**分类（`src/config/navigation.ts`）**：`classes` · `guides` · `items`

**写文章硬规则**（构建时 Zod 校验，不过就 build 失败）：

- `title` ≤ 80 字符；`description` 40–165 字符（**注意 `check-content` 的长度校验比 Zod 宽松，超限只有 build 才报**）
- `category` 必须是 navigation 里的 key
- 正文从 H2 起（H1 由 frontmatter title 渲染）
- 内链必须带尾斜杠（`/classes/all-classes/`）
- 每篇正文 ≥ 3 条内链
- 不确定的数据不写；创作者口播的数值标记为 creator-reported

**封面**：不要手写 `image:` 到 frontmatter —— 直接跑 `pnpm gen-covers`，脚本会生成 1200×675 PNG 并回写 frontmatter（**已有 `image:` 的文章会被当成"用户自备封面"跳过**）。

**常用命令**：

```bash
pnpm check-content      # 内容 lint（frontmatter/内链/长度）
pnpm check-links        # 全站内链审计
pnpm check-config       # 配置一致性（nav/locale/域名）
pnpm check-sitemap      # sitemap 校验（需先 pnpm preview 起服务）
pnpm gen-covers         # 生成封面并写入 frontmatter
pnpm new-post           # 交互式新建文章
pnpm template-audit     # 换皮残留扫描
pnpm submit-indexnow -- --site https://drilltoearthscore.xyz   # 推送 URL 给 Bing/Yandex
```

**数据来源纪律**（本站最重要的一条）：

1. **职业名单/上线日期以 Roblox badge API 为准** —— 每个职业解锁都有独立 badge，创建日期就是上线日期：
   `https://badges.roblox.com/v1/universes/9796898051/badges?limit=100&sortOrder=Asc`（翻页带 `cursor`）
2. **地层/墙 HP/掉落/升级成本**用 Fandom 的 MediaWiki API（网页端 403，API 通）：
   `https://drill-to-earths-core.fandom.com/api.php?action=parse&page=Layers&prop=wikitext&format=json`
3. **新职业、玩家口播数值**用 YouTube 视频描述交叉验证（`yt-dlp --skip-download --print "%(description)s"`）
4. 竞品 `drilltoearthscore.wiki` 数据准确度可参考，但**不要直接抄**；它的职业表缺 12 个职业且无上线日期

**这个游戏没有兑换码**（无 Codes 按钮），所以**不要做 codes 分类/codes 页** —— 这是它和 dungeonlootr / raceforeggs 模板最大的差别。

## 部署（关键：部署 = commit + push）

**Cloudflare Workers Builds（Git 集成）**：push 到 `main` → 自动构建 → 部署。**本地 `wrangler deploy` 只是临时生效，会被下一次 Git 构建覆盖——任何改动必须 commit + push。**

- Build command（dashboard 里配置）：`SITE_URL=https://drilltoearthscore.xyz pnpm run build`
- Deploy command：`npx wrangler deploy`
- 输出目录：`dist/`（由 `wrangler.jsonc` 的 `assets.directory` 指定）

### 待接线清单

| 项 | 状态 |
| --- | --- |
| 域名 `drilltoearthscore.xyz` | ✅ NS 已转 Cloudflare（ariella/seamus） |
| GitHub 仓库 `ken-fs/drilltoearthscore` | ✅ 已推送 |
| Cloudflare Worker + Git 集成 | ✅ 已接通，push → 自动构建 → 自动部署（用 `.well-known/anvilwiki-deploy.txt` 的 commit SHA 验证过） |
| 自定义域 `drilltoearthscore.xyz` | ✅ 已绑定，HTTPS 200，Let's Encrypt 证书已签（至 2026-12-19） |
| **GitHub 仓库变量 `SITE_URL`** | ❌ **必设**，否则 CI `check` job 失败（不影响部署） |
| GitHub 仓库变量 `INDEXNOW_KEY` | ⬜ 可选（不设则 IndexNow workflow 跳过） |
| `www.drilltoearthscore.xyz` | ✅ 已绑定（生产），内容与 apex 完全一致，canonical 指向 apex 防重复内容 |
| GSC 属性 `sc-domain:drilltoearthscore.xyz` | ✅ 已建，服务账号 `gsc-bot@ken-seo-tools` 已加为拥有者；sitemap 已提交（2026-09-21） |
| GA4 `G-FBMCFJEM3S` | ⏳ 代码就位，待 Cloudflare 构建命令加 `PUBLIC_GA_ID` 后生效 |

**线上验收结果（2026-09-20）**：

```
页面健康     17 个 URL 全 200（含 8 篇文章 + 3 分类页 + 法务页）
技术文件     robots / sitemap-index / sitemap-0 / rss / llms.txt / manifest / favicon / IndexNow key 全 200
sitemap      48 条，全部 https://drilltoearthscore.xyz（域名零污染）
robots.txt   Allow: / + Sitemap 指向正确
安全头       HSTS 31536000 / nosniff / SAMEORIGIN / referrer-policy
JSON-LD      Organization + Article + BreadcrumbList + FAQPage(5 题)
浏览器渲染   首页 / all-classes / layers-and-depths 均无报错
部署标记     线上 commit SHA == 本地 HEAD
证书 SAN     drilltoearthscore.xyz + *.drilltoearthscore.xyz（apex 与 www 都覆盖）
```

**排查线上问题的坑（重要）**：本机跑 Clash Verge（mihomo，TUN + fake-ip），会把域名 DNS 劫持成 `198.18.0.x` 假 IP，导致 `dig` / `curl` 全部误报失败 —— 包括连 `dig @1.1.1.1` 都被劫持。**诊断域名问题必须绕过本地 DNS**：

```bash
# 拿真实解析（DoH）
curl -s -H "accept: application/dns-json" "https://cloudflare-dns.com/dns-query?name=drilltoearthscore.xyz&type=A"
# 用真实 IP 绕过本地解析测试
curl -s --resolve "drilltoearthscore.xyz:443:104.21.67.220" "https://drilltoearthscore.xyz/"
# 终极验证：用外部服务器抓（本地代理再坏也不影响结论）
# tavily extract https://www.drilltoearthscore.xyz/classes/
```

**`www` 本地打不开、但线上正常**（2026-09-21 实例）：www 自定义域刚加时，Clash 对该子域的代理路由处于坏状态 —— apex 的 fake IP 能通、www 的 80/443 全挂，且刷新 fake-IP 缓存无效。**站本身没问题**（外部服务器抓取 www 返回 200，内容与 apex 一致）。两处修复：

1. 立即：强制重载 Clash 内核 —— `curl -X PUT --unix-socket /tmp/verge/verge-mihomo.sock "http://localhost/configs?force=true"`（返回 204 即生效）
2. 永久：把自有域名加进 Clash Verge 的 `profiles/Merge.yaml`，走 DIRECT 绕过代理

```yaml
prepend-rules:
  - DOMAIN-SUFFIX,drilltoearthscore.xyz,DIRECT
dns:
  fake-ip-filter:
    - "+.drilltoearthscore.xyz"
```

**两个仓库变量怎么设**（Settings → Secrets and variables → Actions → **Variables** 标签页）：

```
SITE_URL      = https://drilltoearthscore.xyz
INDEXNOW_KEY  = 99483352b40630153d5901e3a14ed160
```

或用 gh CLI（需先 `gh auth login`）：

```bash
gh variable set SITE_URL --body "https://drilltoearthscore.xyz" -R ken-fs/drilltoearthscore
gh variable set INDEXNOW_KEY --body "99483352b40630153d5901e3a14ed160" -R ken-fs/drilltoearthscore
```

**为什么 SITE_URL 必设**：`.github/actions/gates/action.yml` 的 check-config 步骤用 `vars.SITE_URL`，未设时回落到 demo 域名 `https://anvil.wiki`，与 `site.ts` 的 `drilltoearthscore.xyz` 不符 → 报「canonical/og:url/sitemap 会指向错站」并失败。本 fork 用 `wrangler.jsonc` 而非 `.toml`，没有别的回退源。

### 构建命令里的 SITE_URL 不能省（实测过的坑）

```
不带 SITE_URL：  canonical ✅ drilltoearthscore.xyz（来自 site.ts 兜底）
                 sitemap   ❌ https://anvil.wiki/      ← 会把 demo 域名提交给 Google

带 SITE_URL：    canonical ✅ + sitemap ✅ 全部 drilltoearthscore.xyz
```

根因：`astro.config.ts` 的兜底写死 `'https://anvil.wiki'`（上游 demo 域名），与 `src/config/site.ts` 的兜底（`https://${site.domain}`）**不一致**。漏传 SITE_URL 时 canonical 看着是对的，只有 sitemap 静默指向别人家。

**已在本 fork 从根上修掉**（`astro.config.ts` 两处）：兜底改为 `https://${site.domain}`，从 `site.ts` 派生（单一真相源）。验证：不带任何 env 变量构建，sitemap 48 条仍全部指向本站。**这个修复值得推给上游** —— 它把「忘记传 SITE_URL」从静默 SEO 事故降级成无害默认值。

**Cloudflare 侧**（Workers 项目 → 设置 → 构建）：

```
构建命令  SITE_URL=https://drilltoearthscore.xyz PUBLIC_GA_ID=G-FBMCFJEM3S pnpm run build
部署命令  npx wrangler deploy
```

**Google Analytics**：模板的 GA 是**同意门控**的（`BaseLayout` 只定义 `window.__awLoadTrackers`，`CookieConsent` 在访客点接受后才动态注入 gtag）——所以**不要手贴 GA 的原始 snippet**，那样会绕过同意门控。正确做法就是构建命令里的 `PUBLIC_GA_ID=G-FBMCFJEM3S`；不设则整段不渲染、零 JS。

⚠️ **`SITE_URL` 不能写进 `.env`**：`astro.config.ts` 在 Astro 加载 `.env` 之前就执行，`.env` 里的 `SITE_URL` 对 sitemap/site 配置**无效**（只对 `import.meta.env` 的 `PUBLIC_*` 有效）。`PUBLIC_GA_ID` 则**可以**放 `.env`（它走 `import.meta.env`）。

首次推送后的 CI 实况：`e2e-template` ✅ · `ops-toolkit` ✅（typecheck + tests + build 全过）· `check` ❌（就是上面这个 SITE_URL）· `IndexNow` skipped。

### 踩过的坑（从 raceforeggs 移植的修复）

1. **`wrangler.jsonc` 必须在仓库根目录**：wrangler 配置发现会向上找，父目录的 `wrangler.jsonc` 会抢占（`.toml` 输给父目录的 `.jsonc`）。
2. **`pnpm check-config` 第 4 项会报 SITE_URL 缺失**：本 fork 用 `wrangler.jsonc`，脚本只认 `wrangler.toml`。带上 env 跑即可通过：`SITE_URL=https://drilltoearthscore.xyz pnpm check-config`。raceforeggs 同样如此，不影响部署。
3. **构建时环境变量走 build command**：`SITE_URL` / `PUBLIC_GA_ID` / `INDEXNOW_KEY` 都是构建时读取，dashboard 运行时 vars 对静态站无效。
4. **IndexNow key 文件已提交**：`public/99483352b40630153d5901e3a14ed160.txt`，勿删。
5. **模板同步**：`upstream` remote 指向 AnvilWiki（`git fetch upstream && git merge upstream/main`），合并时保留上述 fork 差异。

## 配置速查

| 配置 | 位置 | 当前值 |
| --- | --- | --- |
| 站点标识/域名 | `src/config/site.ts` | Drill to Earth's Core Wiki · drilltoearthscore.xyz |
| 主题色 | `src/styles/globals.css` | `#ea580c` |
| 导航/分类 | `src/config/navigation.ts` | classes / guides / items |
| 首页模块 | `src/locales/en.json` → `home` | hero/start/explore/faq |
| IndexNow | `public/99483352b40630153d5901e3a14ed160.txt` | 已提交 |

## 运营链接

| 项 | 地址 |
| --- | --- |
| 游戏 | https://www.roblox.com/games/101906032112547/Drill-to-Earth-s-Core |
| 线上（临时） | https://drilltoearthscore.493129720ljw.workers.dev |
| GitHub | https://github.com/ken-fs/drilltoearthscore |
| Cloudflare | Workers & Pages → drilltoearthscore（account `70716e073f0925c564bafd0eaf0be307`） |
| GSC 属性 | `sc-domain:drilltoearthscore.xyz`（待接入） |
| 验收 | `node ~/Desktop/david/Ship/scripts/verify.mjs`（待加入基线） |
| 上游模板 | https://github.com/PNGTRID/AnvilWiki |

## 内容清单（2026-09-21 第一批，8 篇）

| 分类 | 页面 | 数据来源 |
| --- | --- | --- |
| classes | all-classes（26 职业 + Core 价格 + 上线日期） | Roblox badge API + Academy 价格表 |
| classes | best-classes（S/A/B/C 强度榜） | 角色定位 + 社区 tier list 交叉验证 |
| classes | reaper（魂球/Soul Lantern/等级门槛） | 竞品站 + 2 个创作者视频一致 |
| guides | layers-and-depths（12 层深度/墙 HP/变异/敌人） | Fandom Layers 页 |
| guides | get-cores（成就奖励 + 宝箱倍率 + boss wall） | Fandom Core/Achievements/Chests 页 |
| guides | reach-100km（三大失败点 + 升级组合） | 地层数据 + 社区共识 |
| guides | beginner-guide（第一小时） | 综合 |
| items | drill-upgrades（5 稀有度全表 + 成本曲线） | Fandom Drill Upgrades 页 |

**竞品格局**：`drilltoearthscore.wiki`（2026-09-12 注册，真实数据但仅 14 职业、无上线日期）+ `drill-to-earths-core.fandom.com`（27 页，10 职业，已过时 6 个月）。我们的优势是**完整 26 职业 + 权威上线日期**。

## 许可

站点代码基于 [AnvilWiki](https://github.com/PNGTRID/AnvilWiki)（MIT）。《Drill to Earth's Core》游戏内容与素材版权归 Game Name. 与 Roblox Corporation 所有；本站为粉丝站，无官方关联。
