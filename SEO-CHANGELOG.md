# SEO 改动台账 · drilltoearthscore.xyz

每次 SEO 调整记一条：为什么、预期、复查日、指标。到期补结果。

## 2026-11-17 · 标题月份改成从核对日期自动算

**为什么**：11-07 扫到、11-17 线上仍是：标题里的月份是手写的，自动更新（autopilot）核对码 / 数据时只改日期字段、按规矩不碰标题，月份就停在上个月。标题月份过期会让人觉得页面没人管（codes / tier list 这类词大家按月搜），也和页面上的核对日期对不上。

**改了什么**
- 移植 game-template-anvil 0d7b70f：frontmatter 认 `{month}`（取 lastModified 的月份）
- /classes/best-classes/：「Drill to Earth's Core Class Tier List (October 2026) — Best Classes」改写成 `({month})`，description / summary 同步，正文「for October 2026」改成「as of the last check」；lastModified 还是 2026-10-17（之后出了 Timekeeper / Spider Queen），线上暂时仍是 October，已交给自动更新引擎复核，改 lastModified 后自动变
- /codes/all-codes/：手写的「(November 2026)」改成 `{month}`（lastModified 11-14，线上不变）

**预期**：以后核对过这页（改核对日期 / lastModified）标题月份自动跟上，不用再手改；每日巡检（seo-structure ⑧）标题月份早于上个月就点名。GSC 里这些页的 CTR 不降，codes / tier list 词的点击有机会小涨。

**复查**：📅 2026-12-01（线上标题是不是 November / December；12-01 后巡检有没有点名这些页；GSC 这几页 28 天 CTR / 点击对比 11-17 前）

**结果**：（待补）
