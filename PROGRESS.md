# PROGRESS — Warhammer 40K《為了帝皇》

> 橫向卷軸動作遊戲，純前端單檔（`index.html`）。手機優先 + PWA。
> 線上：https://rai0603.github.io/warhammer40k/

---

## 專案結構

```
warhammer40k/
├── index.html      ← 唯一檔案（HTML + CSS + JS 全內嵌，~3127 行）
├── PROGRESS.md
└── README.md
```

**單檔架構**：所有邏輯、樣式、PWA manifest、Service Worker、音樂都內嵌在 `index.html`。
改任何東西只動這一個檔；GitHub Pages 從 `main` branch root 自動部署。

## 技術棧

| 項目 | 用法 |
|---|---|
| 渲染 | 原生 Canvas 2D，內部解析度 `VW=480 × VH=270`，等比縮放鋪滿螢幕（`resize()`）|
| 音樂/音效 | **Tone.js**（CDN），程式合成，無音檔。每關一首 8-bar loop + Boss 專曲 |
| PWA | inline blob manifest + Service Worker（離線快取自身），iOS standalone 全螢幕 |
| 操控 | 觸控按鈕（左下移動 / 右下動作鑽石佈局 + 武器切換）+ 鍵盤（`bindBtn`）|
| 版控/部署 | git → GitHub `rai0603/warhammer40k` → GitHub Pages（main root auto-deploy）|

## 遊戲設計現況（v1.0）

### 流程
`TITLE → PLAY ×5 關 → (每關 TRANS 過場) → VICTORY`，死亡進 `GAMEOVER`。狀態機在 `update()` / `state` 變數。

### 關卡（5 關，`level` 0–4）
- 敵血量逐關遞增：`ENEMY_HP_MULT=[1.00,1.10,1.21,1.27,1.33]`
- brute 出現率隨關卡 5%→17%
- 每關尾 Boss：

| 關 | Boss | HP |
|---|---|---|
| 1 | 噬肉巢母 DEVOURER BROODMOTHER | 4500 |
| 2 | 裂顎屠夫 RENDCLAW BUTCHER | 8000 |
| 3 | 瘟疫宿主 PLAGUEBOUND HERALD | 13600 |
| 4 | 影刃潛獵者 SHADOWBLADE STALKER | 24500 |
| 5 | （第 5 Boss，見 `BOSSES[]`）| — |

Boss 三階段（HP >66% / >33% / 其餘）行為切換。

### 武器（拾取「武器晶片」或能量滿 100 升級，上限 LV5）
**遠程 `RANGED[]`**：爆彈槍 BOLTER / 火焰槍 FLAMER（散射）/ 電磁砲 RAILGUN（穿透）/ 槍榴彈 GRENADE（拋射+範圍爆破，射速隨等級加快）/ 充能步槍 CHARGE RIFLE（集氣2秒釋放玩家大小衝擊波，傷害300）
**近戰 `MELEE[]`**：動力劍 POWER SWORD / 動力錘 THUNDER HAMMER（高傷高擊退）/ 動力拳 POWER FIST

### 敵人 `spawnEnemy()`
`crawler / upright / infected / spitter / flyer(飛行)` + `brute`（重裝，會遠程+踩踏）

### 掉落物 `maybeDrop()` / `pickupDrop()`
HP+30 / 能量+40 / 護盾+40 / 武器晶片 / 彈藥強化(180f) / ★帝皇恩賜（無敵+雙倍火力 300f，稀有 1.5%）

### 玩家系統
HP、能量（滿 100 自動升級當前武器）、護盾（上限 60）、無敵幀 `inv`、祝福 `blessing`、分數 `score`。

---

## 程式碼地圖（`index.html` 行號，改動前先定位）

| 區塊 | 行號附近 | 函式 |
|---|---|---|
| PWA / manifest / SW | 99–126 | inline blob |
| 縮放 / 全螢幕 / 暫停 | 137–185 | `resize` `toggleFs` `togglePause` |
| 音樂引擎 | 200–336 | `buildTrack` `initAudio` `setMusic` `sfx` |
| 武器定義 | 339–351 | `RANGED` `MELEE` |
| 關卡 / 生怪 / Boss | 391–456 | `buildLevel` `spawnEnemy` `spawnBoss` `BOSSES` |
| 戰鬥 | 456–541 | `fireRanged` `doMelee` `damageEnemy` `damageBoss` `hurtPlayer` |
| 掉落 / 升級 | 519–535 | `maybeDrop` `checkUpgrade` `pickupDrop` |
| 主迴圈 | 544–714 | `update()` |
| 繪圖 | 715–1035 | `px` `drawMarine` `drawEnemy` `drawBoss` `drawHUD` `drawTitle` 等 |

---

## 待辦 / 點子（TODO）

- [x] 特殊任務關卡 — ✅ 2026-06-01
- [ ] 第 5 Boss 名稱與機制確認 / 補完
- [ ] 補給站：可考慮加入「全補血」高價品、或限購次數
- [x] 分數排行榜（localStorage 本地，存玩家手機）— ✅ 2026-05-30
- [ ] 難度選擇（影響 `ENEMY_HP_MULT` 與生怪密度）
- [ ] 真正的 PNG icon（目前 manifest icon 為程式生成）
- [x] 開場操作教學 / 武器說明 — ✅ 2026-06-05
- [x] 任務終端（選關介面）— ✅ 2026-06-06
- [ ] 觸控手感微調（移動慣性、跳躍判定）

---

## 開發備忘

- **本地預覽**：`python3 -m http.server 8000` 後開 `http://localhost:8000/`（Tone.js / SW 需 http 環境，直接開檔 file:// 部分功能會失效）
- **部署**：`git push` 到 `main` 即自動上線（GitHub Pages），約 1 分鐘生效
- **改完更新此檔**：每次新增功能 / 調平衡，更新上面「現況」與「TODO」

## 變更日誌

- **2026-05-30** — 初版上線：5 關 + 5 Boss、4 遠程 / 3 近戰武器、PWA、Tone.js 配樂。建立 git + GitHub Pages 部署。
- **2026-05-30** — 手機操控優化：左右方向鍵 58→76px、間距加倍；暫停/全螢幕鍵從貼邊改距邊 12%；右下動作叢集整體內縮(跳鍵離開圓角)、射擊↔近戰互換並等大放大為 70px。
- **2026-05-30** — 新增**開場角色選擇 (SELECT 狀態)**：5 軍團各有專屬配色 / 英雄名 / 初期加成；陸戰隊依軍團上色。
  - 極限戰士(藍/提圖斯)：最大生命+30｜血天使(紅/但丁)：近戰起始Lv3｜太空野狼(灰藍/羅根)：護盾+60 生命+10｜暗黑天使(綠/以西結)：爆彈槍起始Lv3｜帝國之拳(金/利西克斯)：電磁砲Lv2 能量+50
  - 操作：點卡片選、再點卡片或「確認出戰」鍵開打；鍵盤 ←→ 選 ENTER 出戰。加成在 `startGame()` 末套用(`LEGIONS[selLegion].apply`)、配色在 `drawMarine()`。
- **2026-05-30** — **本地排行榜**：每場結束 `recordScore()` 把(分數/關卡/軍團/勝負/日期)寫入 `localStorage['wh40k_scores_v1']`(存玩家手機)；GAMEOVER/VICTORY 以 `drawLeaderboard()` 顯示 Top5 + 新紀錄高亮。
- **2026-05-30** — **自動更新機制**：SW 從 inline blob(cache-first)改為獨立 `sw.js`(**network-first**：上線永遠最新、離線備援)。`index.html` 加 `<meta name="build">`，`checkBuild()` 每 2 分鐘比對線上版本，有新版跳「立即更新」橫幅。更新只清舊快取、**不動 localStorage 分數**。

> ⚠️ **部署慣例**：每次改 `index.html` 要 deploy 時，**手動把 `<meta name="build">` 的值改新**（如日期+流水號），否則開著遊戲的玩家不會收到更新提示。`sw.js` 若有改也順手 bump `const C='wh40k-vN'`。目前 build = **2026.06.06.2**。
- **2026-06-05** — **新增任務終端「巢翼暴君 WINGED HIVE TYRANT」**：選角畫面加入第三任務選項「★ 任務終端」；關卡為「虛空獵殺 VOID HUNT」，背景為太空中帝國軍艦對蟲族生物艦的遠方交戰場面（視差捲動艦影＋閃光爆炸）；難度與普通第四關相同（ENEMY_HP_MULT=1.27）；Boss 巢翼暴君HP 18000，三段近戰AI：①低血期 3向爪擊+隨機衝刺②中血期 5向爪擊+必衝刺+召喚飛蟲③激怒期 8向爪雨+持續衝刺；大絕「翼暴咆哮」12向全向翼刃彈+超長衝刺；展翅動畫（Math.sin 翼扇）＋冠刺＋爪繪製；擊倒直接進勝利無補給站。
- **2026-06-06** — **新增任務終端第二關「前線救援 FRONTLINE RESCUE」**：背景同虛空獵殺（太空深處），難度★★★★，長官命令故事：前線友軍告急即刻馳援。關卡中點（x=4200）到達時觸發三名HP剩半的極限戰士加入，作為 `allies[]` 陣列AI盟友一同作戰。最終Boss「蟲族劊子手 TYRANID EXECUTIONER」（HP 20000），三階段AI（鎌爪衝刺/五向+地面衝擊/狂暴連突）＋大絕「執刑之刃」（16向全向彈+三排地面衝擊波）。**全部任務終端關卡結尾改為飛船撤離**（與蟲族侵攻相同），盟友組也會被牽引光束吊升帶走。新增**長官通訊系統**（📡 金色橫幅），每關依里程碑推送對應長官語音訊息。
- **2026-06-06** — **任務終端改為選關介面（TERMINAL 狀態）**：選角畫面點「★ 任務終端」不再直接開始，改為進入獨立選關畫面（state='TERMINAL'）。介面顯示任務卡片（任務名/英文名/目標Boss/描述/難度/HP倍率/Boss剪影），多任務時左右翻頁。目前收錄一個任務「虛空獵殺 VOID HUNT」；未來可直接向 `TERMINAL_MISSIONS[]` 陣列追加任務。鍵盤：←→翻頁、ENTER出發、ESC/Q返回選角；手機：點底部返回、點卡片區翻頁、點按鈕出發。
- **2026-06-05** — **新增教學畫面 TUTORIAL**：選角畫面左下角「[?] 教學」按鈕（或鍵盤任意鍵進入後切換），6頁完整說明：①移動操控（鍵盤+手機+衝刺技巧）②戰鬥技巧（射擊/近戰/換武器/充能步槍/升級/Boss注意）③敵人圖鑑（6種敵人像素圖+攻擊說明+5Boss列表）④掉落物品（6種道具圖示+效果說明）⑤遠程武器（5種附畫面）⑥近戰武器（3種附畫面）。← → 翻頁，右上角✕或ESC返回選角。
- **2026-06-03** — **新增遠程武器「充能步槍 CHARGE RIFLE」**：射速為爆彈槍2/3（rate:21）；按住射擊鍵集氣，2秒後釋放與玩家等大的衝擊波（傷害300，穿透所有敵人，每個只打一次）；HUD 顯示充能進度條，集滿閃白；玩家模型在充能時亮起藍白光圈。按換槍鍵可切換到此武器。
- **2026-06-03** — **Bug fix：特殊任務撤離時主角/盟友消失**：`updateExtract()` 未遞減 `player.inv` / `ally.inv`，導致無敵閃爍計數凍結，約 50% 機率角色永久停在不可見幀。在 `updateExtract()` 開頭補上兩個計數器的每幀遞減。
- **2026-06-01** — **特殊任務關卡「蟲族侵攻 TYRANID ASSAULT」**：選角畫面新增任務選擇列（↑↓/Tab切換，手機點擊），可選正常五關或特殊任務。特殊任務特性：①蟲群侵攻紫暗天空+遠方飛翔蟲群背景 ②關卡進行到一半（x>2600）自動設置電磁塔，塔血量=玩家HP×10，需防禦30秒 ③敵人密度約1.8倍，全程持續波次生怪 ④電腦操控極限戰士同伴(AI)陪同作戰，使用動力劍+爆彈步槍 ⑤最終Boss「焦血毀滅者 BLOODCRUSHER OVERLORD」(HP:24000)，技能組：地面衝擊波/酸液迫擊炮/狂暴連射，大絕「絕命踩踏」。塔血量歸零=失敗；擊敗Boss直接進勝利，不過補給站。
- **2026-05-31** — **關卡間補給站 CAMP 狀態**：擊敗 Boss 後進入「帝國補給站」休息區，背景為夜空中人類雷鷹砲艦對蟲族飛行生物的空戰、帝國金屬壁壘後景。地圖設有 4 個攤位（急救站 500分/護盾站 400分/武器庫 900分/能量站 300分），走到右側發光大門進入下一關；音樂沿用開場選角同款選單主題。build 升為 **2026.05.31.3**。
- **2026-05-30** — **死亡音效**：`gameOver()` 觸發 `deathSfx()`，~2 秒下行哀號(D 小調)+低音下沉+終結重擊延音，用 `Tone.now()` 直接排程(不靠已停的 Transport)。
- **2026-05-30** — **開場/選角選單音樂**：新增 `MENU_SONG`(D 小調進行曲) + `buildMenuTrack()`，`setMusic('menu')`；`loop()` 在 TITLE/SELECT 且 audioReady 時自動播放，進 PLAY 由 `buildLevel()` 切回關卡曲。
  - ⚠️ 教訓：build .3 因 `setMusic` 字串比對沒中、`buildMenuTrack` 沒插進去但 `initAudio` 已呼叫它 → ReferenceError 開不了遊戲；build .4 修復。改字串前務必先讀到「真實」內容。
