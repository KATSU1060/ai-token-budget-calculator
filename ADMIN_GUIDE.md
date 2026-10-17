# AIトークン予算 試算ツール 社内配布ガイド（情報システム部門向け）

このツールは1つのHTMLファイルで完結し、外部と通信しません。配布前に次の3点を設定してください。

## 1. 社内設定を書き換える

`index.html` をテキストエディタで開き、冒頭の `window.AITB_CONFIG` を編集します。

| 項目 | 内容 |
| --- | --- |
| configName | 設定の名前。共有コードを読み込んだとき、作成時の設定と違えば警告が出る |
| defaults | 席の料金（ドル）、為替、API単価、稼働日数などの初期値 |
| annualPrices | 年払いを選んだときの席の料金 |
| accounts | 経費計上の下書きに表示する勘定科目 |
| departments | 共有先の部署と、部署ごとの確認項目 |
| policyNote | 書き出し文面の末尾に入る社内ルール |

## 2. 配布方法を選ぶ

- ファイルで配る（推奨）：共有ドライブなどに置き、各自がダウンロードして開く。Chrome・Edgeの拡張機能は、利用者が個別に許可しない限りローカルファイルを読めない。
- 社内サーバーに置く：サーバーの応答ヘッダーにも同じ通信制限を設定する。
  `Content-Security-Policy: connect-src 'none'; img-src data:; form-action 'none'; frame-src 'none'; object-src 'none'; base-uri 'none'`

## 3. ブラウザ拡張機能を管理する

Chrome・Edge共通の ExtensionSettings ポリシーで、拡張機能を許可制にし、ツールを置くサイトで拡張機能が動かないようにします。

```json
{
  "*": {
    "installation_mode": "blocked",
    "runtime_blocked_hosts": ["*://tools.example.co.jp"]
  },
  "承認済み拡張機能のID": {
    "installation_mode": "allowed",
    "runtime_blocked_hosts": ["*://tools.example.co.jp"]
  }
}
```

- `tools.example.co.jp` はツールを置く社内サーバーのホスト名に置き換える。
- 許可した拡張機能にも `runtime_blocked_hosts` を書いておく。
- 設定は Google 管理コンソール、Microsoft Intune、グループポリシーなど、社内の管理ツールから配布する。反映後、`chrome://policy` または `edge://policy` で確認する。
- ローカルファイルへのアクセスを利用者が拡張機能に許可していないかも、あわせて点検する。

## 4. 部署間の共有の流れ

1. 起案者が試算し、「部署間の共有」で合言葉をつけて共有コードを作る。
2. コードはメールやチャットで、合言葉は電話や別のチャットで渡す。
3. 受け取った部署は、ツールにコードと合言葉を入れて内容を再現し、確認項目にチェックとコメントを入れる。
4. 確認後、新しい共有コードを作って起案者に返す。

共有コードはAES-GCM（256ビット）で暗号化され、合言葉がなければ中身を読めません。合言葉は8文字以上とし、コードと同じ経路で送らないでください。
