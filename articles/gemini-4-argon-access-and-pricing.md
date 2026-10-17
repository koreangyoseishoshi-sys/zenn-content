---
title: "Gemini 4 ArgonがAntigravityに出てこない理由と、料金・使えるプラン"
emoji: "🧪"
type: "tech"
topics: ["AI", "Gemini", "GoogleAI", "LLM", "Antigravity"]
published: false
---

Google の Gemini 4 Argon が、2026年10月13日に一般公開されました。

発表からまだ日が浅く、「どのプランで使えるのか」「100万トークンは入力なのか出力なのか」について、解説記事どうしで書いていることが食い違っています。この記事では、**Google の公式ブログに書かれていることだけ**を整理し、公式で確かめられなかったことは「未確認」として分けて書きます。

（2026年10月14日時点の情報です）

## 結論

- **一般公開は10月13日**（発表は9月30日。最初はサイバー防御の専門家だけに先行提供）
- **Antigravity で使えるのは Google AI Ultra だけ**。Gemini アプリと検索の AI モードは Pro から
- **API は有料枠のみ**。入力 $2・出力 $10（100万トークンあたり）は導入価格で、**2026年11月30日で終わり、その後は $4・$20**
- 「100万トークン」は、英語の発表では**出力の上限**（6.4万から引き上げ）として説明されている

## 提供範囲

日本語の公式ブログ（10月13日）の記載を表にしました。

| 提供先 | 対象 |
|---|---|
| Gemini アプリ | Google AI Ultra・Pro |
| Google 検索の AI モード | Google AI Ultra・Pro |
| Google Antigravity | Google AI Ultra |
| AI Studio・Gemini API | 開発者・企業（**有料枠**） |
| Android Studio | Google AI Ultra（試用） |
| 企業向け | Gemini for business・Gemini API・Antigravity |

開発者向けの発表（10月13日）も、「Gemini API の有料枠で構築を始める」という書き方で、**無料枠には触れていません**。

### Antigravity の一覧に出てこない場合

手元の Antigravity CLI（`agy` 1.3.4・Ultra ではないプラン）でモデル一覧を出すと、10月14日時点で Argon はありませんでした。

```console
$ agy models
gemini-3.8-flash-high	Gemini 3.8 Flash (High)
gemini-3.8-flash-medium	Gemini 3.8 Flash (Medium)
gemini-3.8-flash-low	Gemini 3.8 Flash (Low)
claude-opus-5-5-low	Claude Opus 5.5 (Low)
...
gpt-oss-120b-medium	GPT-OSS 120B (Medium)
```

上の表のとおり、Antigravity は Ultra だけなので、これは想定どおりです。**Gemini アプリでは Pro でも使えるのに、Antigravity では使えない**、という差に注意してください。

## API 料金

| | 導入価格（2026年11月30日まで） | 導入期間の終了後 |
|---|---|---|
| 入力（100万トークンあたり） | $2 | $4 |
| 出力（100万トークンあたり） | $10 | $20 |
| キャッシュされた入力 | 入力価格の95%引き | 同じ |

### 1回あたりの目安

入力10万トークン・出力1万トークンの処理を1回すると、

| | 入力 | 出力 | 合計 |
|---|---|---|---|
| 導入価格 | 0.1M × $2 = $0.20 | 0.01M × $10 = $0.10 | **$0.30** |
| 導入期間の終了後 | 0.1M × $4 = $0.40 | 0.01M × $20 = $0.20 | **$0.60** |

同じ処理が、導入期間が終わると2倍になります。評価や移行の検証をするなら、導入価格のうちに済ませておくのが得です。

## 「100万トークン」は何の上限か

ここが一番混乱しているところです。公式の2つの発表で、書き方が違います。

| 発表 | 書き方 |
|---|---|
| 英語（9月30日） | 出力の上限を、**これまでの64Kトークンから業界最大の1Mトークンに**引き上げた |
| 日本語（10月13日） | Gemini API で「**100 万トークンの統合入力・出力制限**」を提供 |

英語の発表が強調しているのは**出力**です。長いコードや報告書を、途中で切らずに1回で書き切れる、という点が新しいところです。

「100万トークンの長文を読み込める（入力の上限）」と紹介している記事もありますが、**入力だけの上限を示す公式の数字は、この記事の時点では見つけられませんでした**。第三者の価格比較サイトには「コンテキスト1M」と載っているものもありますが、公式の記載ではないので、ここでは採用していません。

## Google が出している評価

いずれも **Google 自身の発表**（9月30日・10月13日）です。

| 評価 | 結果（Google 発表） |
|---|---|
| DeepSWE v1.1 | 77.9%（最高記録） |
| AutomationBench（Zapier） | 51.3%（1位） |
| LVBench | 91.7%（最高記録） |
| CWE-bench v1 | 68%（1位タイ） |
| Vals Index | 1位（数値の記載なし） |
| 間接プロンプトインジェクション（Gray Swan） | 15回の適応的な攻撃で成功率0.7% |

比較相手の数字は、公式ブログでは画像のグラフにしか載っていないため、ここでは転記していません。

## 未確認のこと

公式の発表では確かめられなかったものです。

| 項目 | 状況 |
|---|---|
| API のモデルID | 発表に記載なし。第三者サイトに載っている文字列は公式ではない |
| 入力だけの上限 | 上のとおり、公式の数字を見つけられていない |
| 無料枠 | 公式は「有料枠」とだけ書いている。無料で使えるとの記載はない |

公式の開発者ガイド（AI Studio）が更新されたら、この表を埋めます。

## まとめ

- 使えるかどうかは**プランと場所の組み合わせ**で決まる。Antigravity は Ultra だけ
- API は有料枠のみ。**導入価格は11月30日まで。その後は2倍**
- 「100万トークン」は、公式の英語の発表では**出力の上限**の話
- 解説記事の数字は、公式の発表と突き合わせてから使う

## 参考（公式）

- Google ブログ「Gemini 4 Argon が一般公開されました」（2026年10月13日・日本語）https://blog.google/intl/ja-jp/company-news/technology/introducing-gemini4-argon/
- Google ブログ「Gemini 4 Argon: our next era of frontier intelligence」（2026年9月30日）https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-4-argon/
- Google ブログ「Gemini 4 Argon is now available for developers」（2026年10月13日）https://blog.google/innovation-and-ai/technology/developers-tools/build-with-gemini-4-argon/
