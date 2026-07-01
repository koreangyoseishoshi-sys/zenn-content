---
title: "スプレッドシート自動化の全体像：GAS・VBA・Pythonをどう使い分けるか"
emoji: "📝"
type: "tech"
topics: ["GAS", "Python", "Excel", "スプレッドシート", "自動化"]
published: true
---

## はじめに

スプレッドシート・Excelの自動化を副業や業務改善で活用したいとき、最初に迷うのが「GAS・VBA・Pythonのどれを使うか」だ。

結論から言うと、**GASとPythonは守備範囲が違うだけで優劣ではない**。VBAは「ローカルExcel文化」の産物であり、クラウド時代には用途が限られる。

---

## VBA vs GAS：弱い・強いではなく「守備範囲」の違い

| | VBA | GAS |
|---|---|---|
| 動作環境 | ローカルPC（Excelが必要） | クラウド（ブラウザのみ） |
| 外部API連携 | 難しい（要追加設定） | 簡単（`UrlFetchApp`） |
| 自動実行（タイマー） | PC起動中のみ | サーバー側で常時OK |
| Google系連携 | 不可 | Gmail・Drive・Calendar全部繋がる |

VBAが「弱い」というより、**クラウド時代に設計されていない**のが実態。ローカルのExcelファイルを細かく操作する用途ではVBAは今でも有効だが、外部APIとの連携や自動実行の信頼性ではGASに及ばない。

---

## スプレッドシートはGASだけで完結する

GASだけで以下はすべてカバーできる：

- 外部API取得・定期実行（GSC・GA4の自動取得など）
- Gmail・Drive・Calendar・Formsとの連携
- シート間の集計・転記・フォーマット整形
- Webスクレイピング（`UrlFetchApp`）
- トリガーによるスケジュール自動実行

**スプレッドシートを使う限り、GASで足りないことはほぼない。**

Pythonをわざわざ使う必要が生じるのは以下の限定的な場面だ：

| 場面 | 理由 |
|---|---|
| データ量が数十万行を超える | GASは実行時間に6分制限がある |
| 機械学習・高度な統計処理が必要 | GASにはそのライブラリがない |
| スプレッドシート以外のシステム（DB・Excel）と横断処理したい | 環境をPythonで統一する方が楽 |

---

## PythonとExcelの繋ぎ方（3パターン）

Excelが絡む案件でPythonを使う場合、繋ぎ方は主に3パターンある。

### パターン1：ファイル読み書き（`openpyxl` / `pandas`）

最も一般的。PythonがバックグラウンドでExcelファイルを生成・更新する。

```
データ取得（API・DB・CSV）
   ↓ Python（pandas）で加工
   ↓ openpyxlでExcel書き出し
→ 完成した.xlsxを配布・送付
```

**よくある用途**：毎月の売上集計・請求書自動生成・複数シートの統合

### パターン2：Excelを直接操作（`xlwings`）

ExcelアプリをPythonから遠隔操作する。VBAの代替として使える。

```python
import xlwings as xw
wb = xw.Book('report.xlsx')
ws = wb.sheets['集計']
ws['A1'].value = '自動入力'
```

VBAと同じ感覚でExcelを操作しつつ、pandasやrequestsなどPythonの外部ライブラリも使えるのが強み。

### パターン3：Python関数をExcel内で呼び出す（`xlwings` UDF）

ExcelのセルからPython関数を直接呼ぶ。GASのカスタム関数に相当。高度な分析処理をExcel内に組み込める。

---

## 副業・業務改善での使い分け判断

```
案件の舞台がスプレッドシート
  → GAS 一択
  理由：Google Workspaceが無料・サーバー不要・すぐ動く

案件の舞台がExcel（社内システム・Excel文化の会社）
  → Python + openpyxl / xlwings
  理由：VBAの限界（外部API・データ量）をPythonで突破できる

大規模・定期バッチ系
  → Python + クラウド（AWS Lambda等）でExcel生成・配布
```

---

## 学習の優先順位

1. **GASを深める**：学習コストが低く、すぐ案件になる。スプレッドシート×API連携が主戦場
2. **Python `pandas` + `openpyxl`**：データ加工→Excel出力のパターンを習得。Excel文化の企業への提案力が上がる
3. **`xlwings`**：既存のExcel運用を壊さずにPythonを組み込む案件向け

VBAは新たに覚える必要はほぼなく、「VBAで作られた既存ファイルをPythonに移植する」案件を取る場合に読める程度で十分。

---

## まとめ

| ツール | 主な用途 |
|---|---|
| **GAS** | スプレッドシート自動化・Google系API連携・定期実行 |
| **VBA** | ローカルExcelの細かい操作（レガシー案件のみ） |
| **Python（openpyxl/pandas）** | Excelファイルの生成・加工・大量データ処理 |
| **Python（xlwings）** | ExcelアプリをPythonで直接操作・VBAの代替 |

スプレッドシート案件はGAS、Excel案件はPythonと割り切ると守備範囲が明確になる。

## 作業ログ

### 2026-07-01T13:13:37.713Z Codex Zenn投稿

- Zenn連携リポジトリへ同期し、commitまで完了。GitHub Desktopでpush待ち。
- target: /Users/icloud/zenn-content/articles/spreadsheet-automation-gas-vba-python.md
