# PUBLISHING — venera-unified-configs 发版流程

自托管书源仓（unified B4）。unified 通过 `SourceRepositories.parseCatalog`
消费这两份 catalog；任何手写 JSON 都必须先过真实解析再打 tag。

## 布局

```
catalog/index.json      默认库（稳定源，新安装预置，老用户迁移自动接入）
catalog/optional.json   可选库（新源/实验源，用户手动添加仓库后安装）
sources/*.js            书源脚本（只放 .js，不放 probe/bench .py）
```

当前 tag：`sources-v1`（jsDelivr）。仓库地址形态：

- 默认库：`https://cdn.jsdelivr.net/gh/ACGNnsj/venera-unified-configs@sources-v1/catalog/index.json`
- 可选库：`https://cdn.jsdelivr.net/gh/ACGNnsj/venera-unified-configs@sources-v1/catalog/optional.json`
- 脚本：`https://cdn.jsdelivr.net/gh/ACGNnsj/venera-unified-configs@sources-v1/sources/<file>.js`

## 铁律

1. **URL 全 pin 到 tag**：catalog 条目 `url` 与脚本内自引用 `url` 一律带
   `@sources-N`，永不指向 `master`/`main` 浮动分支，不出现个人仓地址，
   路径中不出现 `%20`（目录名不用空格）。
2. **版本三处一致**：`sources/<key>.js` 内的 `version` ＝ 对应 catalog 条目
   的 `version`。不一致会导致误报更新或漏报更新。
3. **官方 URL 不动**：脚本内自引用 `url` 若指向官方
   `venera-app/venera-configs`（如 copy_manga/baozi/hitomi/manhuagui），
   保持原样，只换个人仓（lll152872）地址。
4. **默认库只收稳定源**：新源先放 `optional.json` 观察至少一个版本周期，
   无分类页/章节序等硬伤后再进 `index.json`。`ManHuaGui` 的 key 大小写
   与文件名 `manhuagui.js` 不同，改动时注意对应。
5. **`key` 必须匹配 `^\w+$`，`version` 必须匹配
   `^\d+\.\d+\.\d+(?:[.\-].+)?$`**，否则 `parseCatalog` 整条 skip；
   全 skip 会抛 `emptyCatalog`，仓库直接不可用。

## 发新版（改脚本或改版本）

1. 改 `sources/<file>.js`，同步 bump 其 `version` 与 catalog 条目 `version`。
2. 本地校验：
   - `python3 -c "import json; json.load(open('catalog/index.json')); json.load(open('catalog/optional.json'))"`
   - 在 unified 仓跑 `flutter test test/features/comic_source/source_selfhost_test.dart`
    （含真实文件解析用例 `real configs checkout parses`，会读本仓 sibling 路径）。
3. `git add -A && git commit -m "sources: <说明>" && git push`
4. 若脚本内容或 catalog 变化需要用户可见，**打新 tag**（如 `sources-v2`），
   并把本仓所有 `@sources-N` URL 与 unified 侧
   `lib/foundation/comic_source_selfhost.dart` 的 `kSelfhostTag` 同步到新 tag，
   再跑一遍上述单测后 push tag：
   `git tag sources-v2 && git push origin sources-v2`
   - 只改 `PUBLISHING.md`/注释不打新 tag。
   - 旧 tag 永不删除、不 force-push（已装客户端 pinned 在旧 tag 上）。

## 加新源

1. 把 `.js` 放入 `sources/`，自引用 `url` 写自托管 pin 版。
2. 先加进 `catalog/optional.json`（见铁律 4），`fileName` 与磁盘文件名一致。
3. 跑第 2 步校验，确认 `parseCatalog` 报 `entries+1、skipped 0`。
4. commit + push；需要用户可见则按上节打新 tag。

## unified 侧联动

- 默认仓库地址与 tag 常量在
  `unified/lib/foundation/comic_source_selfhost.dart`
 （`kSelfhostCatalogUrl` / `kSelfhostTag`）；tag 变化必须同步改这里并发版。
- 已装客户端的仓库地址存在本地设置里，**换 tag 不会自动跟随**，
  需随 app 发版带迁移或提示用户手动改仓库 URL。
- `jm.js` 的自引用 URL 即 A13 遗留项，已在 `sources-v1` 换成自托管 pin 版。
