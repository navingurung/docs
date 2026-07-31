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

## 13. customers/redact をテストする
同じ手順で Topic を `customers/redact` にして実行する。

確認する場所:
- ngrok inspector
- バックエンドのログ — `[compliance] customer redact for shop=...`

## 14. shop/redact をテストする（最後に実行）
同じ手順で Topic を `shop/redact` にして実行する。

これはデータを実際に削除するので、他の2つが終わった後、最後にテストする。

確認する場所:
- ngrok inspector
- バックエンドのログ — `[compliance] shop redact completed for shop=...`
- DB上で `shopify_stores` の該当行が削除されているか、紐づく `shops` の shopify関連カラムがクリアされているか
