# Shopify App - shopify.app.toml アクセス・編集・デプロイ・テスト手順

## 1. フォルダを作成する
例:
```
mkdir taimatsu-tax-free-shopify-cli
```

## 2. そのフォルダに移動する
```
cd taimatsu-tax-free-shopify-cli
```

## 3. CLIをリンクする
```
shopify app config link
```

## 4. Organizationを選択する
```
?  Which organization do you want to use?

   TAIMATSU Co., Ltd. (27121667)
>  TAIMATSU Co., Ltd. (225489373)
   testing (223227643)

   Press ↑↓ arrows to select, enter to confirm.
```

## 5. 新規作成 or 既存アプリに接続を選択する
```
?  Create this project as a new app on Shopify?

   (y) Yes, create it as a new app
>  (n) No, connect it to an existing app
```

## 6. 対象のアプリを選択する
```
?  Which existing app is this for?

>  st-sandbox
   SAMURAI TAX

   Press ↑↓ arrows to select, enter to confirm.
```

## 7. 成功メッセージを確認する
```
╭─ success ─────────────────────────────────────────────────────────╮
│                                                                    │
│  shopify.app.toml is now linked to "st-sandbox" on Shopify         │
│                                                                    │
│  Using shopify.app.toml as your default config.                    │
│                                                                    │
│  Next steps                                                        │
│    • Make updates to shopify.app.toml in your local project        │
│    • To upload your config, run `shopify app deploy`               │
│                                                                    │
╰─────────────────────────────────────────────────────────────────────╯
```

## 8. ファイルができていることを確認する
```
➜  taimatsu-tax-free-shopify-cli ls
shopify.app.toml
```

## 9. 中身を確認する
```
➜  taimatsu-tax-free-shopify-cli cat shopify.app.toml 
# Learn more about configuring your app at https://shopify.dev/docs/apps/tools/cli/configuration

client_id = "f0ad5125d8b3278a214343977803f075"
application_url = "https://simple-plentiful-puzzling.ngrok-free.dev/shopify/app"
embedded = true
name = "st-sandbox"

[webhooks]
api_version = "2026-07"

  [[webhooks.subscriptions]]
  uri = "https://simple-plentiful-puzzling.ngrok-free.dev/shopify/webhooks/compliance"
  compliance_topics = [ "customers/data_request", "customers/redact", "shop/redact" ]

  [[webhooks.subscriptions]]
  uri = "https://simple-plentiful-puzzling.ngrok-free.dev/shopify/webhooks/app/uninstalled"
  topics = [ "app/uninstalled" ]

[access_scopes]
# Learn more at https://shopify.dev/docs/apps/tools/cli/configuration#access_scopes
scopes = "read_locations,read_orders"
optional_scopes = [ ]
use_legacy_install_flow = false

[auth]
redirect_urls = [ ]
```

## 10. VS Codeで開いて編集する
webhookが未設定の場合は、`[[webhooks.subscriptions]]` ブロックを追加する。

例 (app/uninstalled を追加する場合):
```toml
  [[webhooks.subscriptions]]
  uri = "https://<your-tunnel-url>/shopify/webhooks/app/uninstalled"
  topics = [ "app/uninstalled" ]
```

保存する。

## 11. デプロイする
```
shopify app deploy
```
変更内容を確認 → `y` (Yes, release this new version) を選択。
新しいバージョンがShopify側にリリースされる（例: `st-sandbox-6`）。

---

# ここから: 3つのWebhookをテストする

## なぜこれが必須なのか

Shopifyの公開アプリ（Public App）は、`customers/data_request`・`customers/redact`・`shop/redact` の3つのGDPR compliance webhookに対応することが**必須**。これは、アプリが実際に顧客データを保存しているかどうかに関わらず、`read_orders` のように顧客・注文データへのアクセス権を持つスコープを1つでも申請している時点で課される要件。

Shopifyの審査チームは申請時にこの3つのwebhookへ実際にテストイベントを送信し、正しく200系レスポンスを返すか確認する。ここが未実装・不正な場合、審査に通らない最も一般的な理由の一つとなる。

`app/uninstalled` はGDPR項目ではないが、ストアごとのアクセストークンのライフサイクル管理（アンインストール時に無効化する）のために標準的に必要。

## 12. customers/data_request をテストする
```
shopify app webhook trigger
```
- Webhook ApiVersion → `2026-07`
- Webhook Topic → `customers/data_request`
- Delivery method → `HTTP`
- Address for delivery → `https://simple-plentiful-puzzling.ngrok-free.dev/shopify/webhooks/compliance`

成功時:
```
✅ Success! Webhook has been enqueued for delivery.
```

確認する場所:
- ngrok inspector (`http://127.0.0.1:4040`) — リクエストが届いているか
- バックエンドのログ — `[compliance] data request for shop=...` が出ているか
  
<img width="1296" height="783" alt="Screenshot 2026-07-31 at 17 17 01" src="https://github.com/user-attachments/assets/622e2ac7-8876-4ab3-86f4-3a7d68553e81" />

## Real Example
```bash
➜  taimatsu-tax-free-shopify-cli shopify app webhook trigger
?  Webhook ApiVersion:
✔  2026-07

?  Webhook Topic:
✔  customers/data_request

?  Delivery method:
✔  HTTP

?  Address for delivery:
✔  https://simple-plentiful-puzzling.ngrok-free.dev/shopify/webhooks/compliance

╭─ info ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                 │
│  Using shopify.app.toml for default values:                                                                                                                                     │
│                                                                                                                                                                                 │
│    • App:             st-sandbox                                                                                                                                                │
│                                                                                                                                                                                 │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯

✅ Success! Webhook has been enqueued for delivery.
```

<img width="1270" height="311" alt="Screenshot 2026-07-31 at 17 18 05" src="https://github.com/user-attachments/assets/5a508fd0-ce4e-46e2-8cd0-c963b2b4b4d4" />

**結果: ✅ 成功 — ngrok inspectorで200 OK、バックエンドログに `[compliance] data request for shop=...` を確認。**

## 13. customers/redact をテストする
同じ手順で Topic を `customers/redact` にして実行する。

確認する場所:
- ngrok inspector
- バックエンドのログ — `[compliance] customer redact for shop=...`

**結果: ✅ 成功 — ngrok inspectorで200 OK、バックエンドログに `[compliance] customer redact for shop=...` を確認。**

<img width="1301" height="388" alt="Screenshot 2026-07-31 at 17 19 35" src="https://github.com/user-attachments/assets/424f43e6-8ab8-4cc6-aefe-7d0b1e5a8080" />


## 14. shop/redact をテストする（最後に実行）
同じ手順で Topic を `shop/redact` にして実行する。

これはデータを実際に削除するので、他の2つが終わった後、最後にテストする。

確認する場所:
- ngrok inspector
- バックエンドのログ — `[compliance] shop redact completed for shop=...`
- DB上で `shopify_stores` の該当行が削除されているか、紐づく `shops` の shopify関連カラムがクリアされているか


<img width="1297" height="390" alt="Screenshot 2026-07-31 at 17 22 22" src="https://github.com/user-attachments/assets/c7c60656-e693-42a7-a36e-f4498c615caf" />

**結果: ✅ 成功 — `shopify_stores` テーブルから該当行が削除され、紐づく `shops` の `shopify_store_id` / `shopify_location_id_public` / `use_shopify_public` がすべてクリアされたことを確認。**

---

# テスト完了サマリー

AdminBackend側の全10エンドポイントを実際のShopify開発ストアに対してテストし、すべて正常動作を確認した。

| # | エンドポイント | 結果 |
|---|---|---|
| 1 | `GET /shopify/app` | ✅ |
| 2 | `POST /shopify/auth/token` | ✅ |
| 3 | `POST /shopify/link-shopify-store` | ✅ |
| 4 | `POST /shopify/unlink-shopify-store` | ✅ |
| 5 | `GET /shopify/stores/available-to-connect` | ✅ |
| 6 | `POST /shopify/webhooks/app/uninstalled` | ✅ |
| 7 | `POST /shopify/webhooks/compliance`（GDPR 3件） | ✅ |
| 8 | `GET /shopify/stores/{store_id}/locations`（GraphQL） | ✅ |
| 9 | `POST /shopify/shop/connect` | ✅ |
| 10 | `DELETE /shopify/shop/disconnect` | ✅ |

