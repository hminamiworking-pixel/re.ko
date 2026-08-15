# Re：居（中古ブランド品の仕入れ・出品管理アプリ）

## このリポジトリについて
- `index.html` の**単一ファイル**で完結するアプリ（HTML + CSS + JavaScript）
- ビルド不要。`index.html` を直接編集する
- バックエンドは Supabase（プロジェクトref: `pnulmigiszlowqohwfxy`）
- 認証あり。ログインしたユーザーだけがデータを読み書きできる（RLS）

## 公開先
- 本番（クライアント配布用）: https://re-ko.netlify.app
- 予備: https://hminamiworking-pixel.github.io/re.ko/
- **main に push すると GitHub Actions が Netlify へ自動デプロイする**（`.github/workflows/netlify.yml`）

## 作業の進め方

### 1. 修正する
`index.html` を編集する。主要な箇所の目印:
- 画面: `<div id="dashboard">` `<div id="register">` `<div id="products">` `<div id="sales">`
- 商品編集モーダル: `id="editModal"`（STEP1 仕入れ / STEP2 出品準備 / STEP3 出品）
- AI生成: `function generateAI()` / トレンド検索: `function fetchTrend()`
- 集計: `buildSheetTable()` `buildStockTable()` `buildCohortTable()`
- 定数: `AI_MODEL` `AI_MAX_IMAGES` `FEE_MAP` `ALL_STATUSES` `SHAPE_MAP` `SIZE_FIELDS`

### 2. 必ず構文チェックする（ビルドがないため、これが唯一の防波堤）
```bash
python3 -c "
import re;html=open('index.html',encoding='utf-8').read()
open('/tmp/chk.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>',html,re.S)[-1])"
node --check /tmp/chk.js && echo "SYNTAX OK"
```
**SYNTAX OK が出なければ絶対に push しない。** 構文エラーのまま出すとアプリ全体が動かなくなる。

### 3. デプロイ
`git push origin main` するだけで本番に反映される。

## 重要なルール

### デプロイ前に必ずユーザーの承認を取る
Netlify のデプロイ回数に上限があるため、**勝手に push しない**。
修正内容を説明し、「デプロイしていいか」を聞いてから push する。

### 出先（スマホ）からの依頼で特に注意すること
スマホからの指示では画面での目視確認ができない。そのため:
- **安全な修正のみ実施する**: 文言・ラベルの変更、表示/非表示、選択肢の追加、色やサイズなどの見た目
- **避けるべき修正**: 金額計算、利益・在庫の集計ロジック、保存処理、認証まわり
  → これらは「戻ってから確認込みで対応しましょう」と提案する
- 判断に迷う依頼は勝手に解釈せず、必ず質問する

### 壊れたときの戻し方
```bash
git revert HEAD --no-edit && git push origin main
```
直前のデプロイを取り消して、1つ前の正常な状態に戻せる。

## 金額の定義（変更しないこと。過去に何度も調整して確定した仕様）
- 利益①（メインの利益 `profit_gross`）＝ 売上 ×(1−手数料) − 原価 − 送料
- 利益②（`profit_after_shipping`）＝ 売上 ×(1−手数料) − 送料
- 利益率 ＝ 利益① ÷ 売上
- 売掛金 ＝ 売上 ×(1−手数料) − 送料（ステータスが「発送前」「受け取り評価前」のみ）
- 在庫回転率 ＝ 売却点数 ÷ 平均在庫点数
- 在庫の判定は日付ベース（出品日 ≤ その日 かつ 未売却）。金額は下振れ値（`price_low`）

## 集計の基準日
- スプレッドシートは**仕入れ日（`created_at`）基準**。7/1に仕入れて7/25に売れた場合も、売上は7/1の週に計上する

## 依頼の受付窓口
クライアントからの要望は Lark のフォームに届く。
Base: https://qjpyd73bnvwh.jp.larksuite.com/base/DW9qbboMFaO7hgscmMVjLqKippc
（app_token: `DW9qbboMFaO7hgscmMVjLqKippc` / table_id: `tblGyqjI2FKA3rJ7`）
対応したら、そのレコードの「対応状況」と「対応メモ」を更新する。

## DB（Supabase）を変更する場合
カラム追加などが必要になったら、ローカル環境（Mac）で Supabase CLI を使う必要がある。
クラウド環境からは実行できないため、その場合は**ユーザーに「戻ってから対応が必要」と伝える**。
