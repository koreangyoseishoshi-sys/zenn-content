---
title: "GASでGSC・GA4を毎日自動取得してスプレッドシートに蓄積する──手動アドイン運用からの脱却"
emoji: "📊"
type: "tech"
topics: ["GAS", "GoogleAnalytics", "SearchConsole", "自動化", "ブログ運営"]
published: true
---

## はじめに

ブログやサイトを運営していると、定期的にGSC（Google Search Console）とGA4（Google Analytics 4）のデータを確認したくなる。

よくある運用がこれだ。

- Spreadsheetのアドイン（Search Analytics for Sheets など）を使って手動でエクスポート
- 気が向いたときだけ取得するので、データが飛び飛びになる
- 毎回同じ操作を繰り返すのが地味につらい

これを **Google Apps Script（GAS）で完全自動化**する。毎朝9時に前日のデータが自動でSpreadsheetに追記され、分析したいときはいつでもデータが揃っている状態を作る。

---

## アーキテクチャ

```
[毎日AM9:00 タイマートリガー]
        ↓
[GAS: runDailyAnalyticsFetch()]
    ↓               ↓
Search Console    GA4 Data
   API v3          API
    ↓               ↓
[Spreadsheet: spacelink_analytics_log]
  ├── GSC_daily シート（date / query / page / clicks / impressions / ctr / position）
  └── GA4_daily シート（date / pagePath / sessions / pageViews / engagementRate / newUsers）
```

シートは `GSC_daily` と `GA4_daily` の**固定2枚**。毎日、末尾に行を追記していく。日付列で管理するので、期間を絞った分析も容易にできる。

シートを日付ごとに増やす方式も検討したが、1年で365シートになりファイルが重くなるためやめた。行追記方式ならGoogleスプレッドシートの上限（最大1,000万セル）まで数十年は問題ない。

---

## 必要な準備

### 1. APIの有効化

Google Cloud Console（[console.cloud.google.com](https://console.cloud.google.com/)）で以下を有効化する。

- **Google Search Console API**
- **Google Analytics Data API**

GASからのアクセスは「高度なサービス」として追加するため、Cloud Consoleでの有効化が必須。

### 2. GA4のプロパティID確認

GA4管理画面 → 管理 → プロパティ設定 → **プロパティID**（数字10桁程度）をメモしておく。

---

## GASスクリプト

GASエディタ（[script.google.com](https://script.google.com/)）で新規プロジェクトを作成し、以下を貼り付ける。

```javascript
// ========================================
// 定数定義：ここを自分の環境に合わせて記入
// ========================================
const SITE_URL     = 'https://your-site.com/'; // TODO: GSCプロパティURLに変更
const PROPERTY_ID  = 'YOUR_GA4_PROPERTY_ID';   // TODO: GA4プロパティIDを記入
const SPREADSHEET_ID = '';                       // TODO: 初回実行後に自動設定される（空のままでOK）

// ========================================
// メイン関数：タイマートリガーから呼ぶ
// ========================================
function runDailyAnalyticsFetch() {
  const ss = getOrCreateSpreadsheet_();
  const yesterday = getYesterday_();

  appendGscRows_(ss, yesterday);
  appendGa4Rows_(ss, yesterday);
}

// ========================================
// GSCデータ取得・追記
// ========================================
function appendGscRows_(ss, date) {
  const sheet = ss.getSheetByName('GSC_daily');
  
  // 重複チェック：同じdateが既にあればスキップ
  const existing = sheet.getDataRange().getValues();
  if (existing.some(row => row[0] === date)) return;

  const response = UrlFetchApp.fetch(
    `https://searchconsole.googleapis.com/webmasters/v3/sites/${encodeURIComponent(SITE_URL)}/searchAnalytics/query`,
    {
      method: 'POST',
      headers: { Authorization: `Bearer ${ScriptApp.getOAuthToken()}` },
      contentType: 'application/json',
      payload: JSON.stringify({
        startDate: date,
        endDate: date,
        dimensions: ['query', 'page'],
        rowLimit: 500
      })
    }
  );

  const data = JSON.parse(response.getContentText());
  if (!data.rows) return;

  const rows = data.rows.map(r => [
    date,
    r.keys[0],          // query
    r.keys[1],          // page
    r.clicks,
    r.impressions,
    r.ctr,
    r.position
  ]);
  sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, 7).setValues(rows);
}

// ========================================
// GA4データ取得・追記
// ========================================
function appendGa4Rows_(ss, date) {
  const sheet = ss.getSheetByName('GA4_daily');
  
  const existing = sheet.getDataRange().getValues();
  if (existing.some(row => row[0] === date)) return;

  // GA4 Data API（高度なサービスが必要）
  const report = AnalyticsData.Properties.runReport(
    {
      dateRanges: [{ startDate: date, endDate: date }],
      dimensions: [{ name: 'pagePath' }],
      metrics: [
        { name: 'sessions' },
        { name: 'screenPageViews' },
        { name: 'engagementRate' },
        { name: 'newUsers' }
      ]
    },
    `properties/${PROPERTY_ID}`
  );

  if (!report.rows) return;

  const rows = report.rows.map(r => [
    date,
    r.dimensionValues[0].value,   // pagePath
    r.metricValues[0].value,      // sessions
    r.metricValues[1].value,      // pageViews
    r.metricValues[2].value,      // engagementRate
    r.metricValues[3].value       // newUsers
  ]);
  sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, 6).setValues(rows);
}

// ========================================
// 初回セットアップ：Spreadsheet作成
// ========================================
function setupSpreadsheetOnly() {
  const ss = SpreadsheetApp.create('spacelink_analytics_log');

  const gsc = ss.getActiveSheet();
  gsc.setName('GSC_daily');
  gsc.appendRow(['date', 'query', 'page', 'clicks', 'impressions', 'ctr', 'position']);

  const ga4 = ss.insertSheet('GA4_daily');
  ga4.appendRow(['date', 'pagePath', 'sessions', 'pageViews', 'engagementRate', 'newUsers']);

  Logger.log('Spreadsheet ID: ' + ss.getId());
  // ログに表示されたIDを SPREADSHEET_ID 定数に貼り付ける
}

// ========================================
// タイマートリガー設定
// ========================================
function installDailyTrigger() {
  ScriptApp.newTrigger('runDailyAnalyticsFetch')
    .timeBased()
    .atHour(9)
    .everyDays(1)
    .inTimezone('Asia/Tokyo')
    .create();
  Logger.log('トリガー設定完了');
}

// ========================================
// ユーティリティ
// ========================================
function getYesterday_() {
  const d = new Date();
  d.setDate(d.getDate() - 1);
  return Utilities.formatDate(d, 'Asia/Tokyo', 'yyyy-MM-dd');
}

function getOrCreateSpreadsheet_() {
  if (SPREADSHEET_ID) return SpreadsheetApp.openById(SPREADSHEET_ID);
  // IDが未設定の場合は先に setupSpreadsheetOnly() を実行すること
  throw new Error('SPREADSHEET_IDが未設定です。setupSpreadsheetOnly()を先に実行してください。');
}
```

---

## セットアップ手順

### Step 1：GASプロジェクト作成

1. [script.google.com](https://script.google.com/) → 「新しいプロジェクト」
2. 上記スクリプトを貼り付け
3. `SITE_URL` と `PROPERTY_ID` を自分の値に書き換える

### Step 2：高度なサービスの追加

GASエディタ左メニュー「サービス」→「Google Analytics Data API」を追加（バージョン v1beta）。

### Step 3：Spreadsheetを初回作成

`setupSpreadsheetOnly` 関数を選択して「実行」。ログに表示された **Spreadsheet ID** を `SPREADSHEET_ID` 定数に貼り付ける。

### Step 4：OAuth認証

`runDailyAnalyticsFetch` を選択して「実行」→ 権限の確認が求められるので「許可」。これで初回OAuth認証が完了する。

### Step 5：タイマートリガー設定

`installDailyTrigger` を選択して「実行」。毎日AM9:00（JST）に自動実行されるようになる。

---

## 実装のポイント

### 重複チェックで冪等性を保つ

手動で再実行したとき、同じ日付のデータが二重に追記されないよう、既存データに同じ `date` があればスキップする処理を入れている。手動テストや再実行を安心して行える。

### GSCのデータ遅延に注意

GSCのデータは**2〜3日の遅延**がある。前日データを取得しても実際には2日前のデータが返ることがある。分析時はこの点を念頭に置く。

### GA4 Data APIは「高度なサービス」が必要

GA4のデータ取得は `UrlFetchApp` で直接叩く方法もあるが、GASの「高度なサービス」として `AnalyticsData` を追加する方がコードがシンプルになる。Cloud ConsoleでのAPI有効化も忘れずに。

---

## 運用イメージ

```
spacelink_analytics_log.xlsx
├── GSC_daily
│   date       | query           | page            | clicks | ...
│   2026-06-28 | ビザ 更新       | /zairyu-koshin/ | 12     | ...
│   2026-06-29 | 在留カード 更新 | /zairyu-koshin/ | 15     | ...
│   ← 毎日末尾に追記
│
└── GA4_daily
    date       | pagePath        | sessions | pageViews | ...
    2026-06-28 | /zairyu-koshin/ | 45       | 67        | ...
    2026-06-29 | /zairyu-koshin/ | 51       | 73        | ...
```

このSpreadsheetをGeminiやClaude Opusに渡して「先週の検索パフォーマンスを分析して」と聞くだけで、AI駆動の定期レポートが実現できる。

---

## まとめ

| 項目 | 変更前（手動） | 変更後（GAS自動化） |
|---|---|---|
| 取得頻度 | 気が向いたとき | 毎日AM9:00 自動 |
| データの継続性 | 飛び飛び | 全日分が蓄積 |
| 作業コスト | 毎回5〜10分 | 初回セットアップのみ |
| AI分析との連携 | 毎回手動エクスポート | URL1本を渡すだけ |

初回セットアップは30分程度。一度動かせば、あとは完全に放置でデータが蓄積されていく。ブログ運営の計測を習慣化したい人にとって、最も費用対効果の高い自動化の一つだと思う。
