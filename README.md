# go-zaim

Zaim API向けのGo SDKです。OAuth 1.0aによる認証と、家計簿・ユーザー情報・マスターデータの操作を提供します。外部ライブラリへの依存はありません。

Go 1.26.2以上が必要です。Goパッケージ名は`zaim`、モジュールパスは`github.com/yone-k/go-zaim`です。

## 使い方

取得済みのOAuth認証情報を渡してクライアントを作成します。認証情報の保存や設定ファイルの読込みは、呼出し側で行ってください。

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"

	"github.com/yone-k/go-zaim"
)

func main() {
	client := zaim.New(zaim.OAuthConfig{
		ConsumerKey:       os.Getenv("ZAIM_CONSUMER_KEY"),
		ConsumerSecret:    os.Getenv("ZAIM_CONSUMER_SECRET"),
		AccessToken:       os.Getenv("ZAIM_ACCESS_TOKEN"),
		AccessTokenSecret: os.Getenv("ZAIM_ACCESS_TOKEN_SECRET"),
	})

	user, err := client.VerifyAuth(context.Background())
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%+v\n", user)

	body, err := client.Request(context.Background(), http.MethodGet, "/v2/home/money", map[string]string{
		"limit": "20",
		"page":  "1",
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(body))
}
```

`Request`はAPI応答を`json.RawMessage`で返します。小数金額や、SDKの型に含まれないフィールドを保持したい場合に使います。GET・POST・PUT・DELETEを受け付け、APIパスは`/v2/home/money`のような相対パスで指定します。

型付きの取得・作成・更新APIも利用できます。型付きAPIの金額は`int`です。作成・更新・削除メソッドは`error`を返し、成功時の応答本文は返しません。応答本文が必要な場合は`Request`を使ってください。

## API

| 対象 | 主なAPI |
|---|---|
| クライアント | `New`、`NewWithOptions`、`ClientOptions` |
| OAuth認証 | `RequestToken`、`GetAuthorizeURL`、`ExchangeAccessToken` |
| ユーザー | `VerifyAuth` |
| 家計簿 | `ListMoney`、`CreatePayment`、`CreateIncome`、`CreateTransfer`、`UpdateMoney`、`DeleteMoney` |
| マスター | `ListUserCategories`、`ListDefaultCategories`、`ListUserGenres`、`ListDefaultGenres`、`ListUserAccounts`、`ListCurrencies` |
| 生JSON取得 | `Request` |
| HTTPエラー | `HTTPError`の`StatusCode`・`Body` |

`NewWithOptions`では`BaseURL`と`HTTPClient`を指定できます。テスト用サーバーへの接続や、HTTPクライアントの設定に使います。受け取った`context.Context`はHTTP通信まで渡し、自動再試行は行いません。

## 開発・検証

```bash
GOWORK=off go test -race ./...
GOWORK=off go vet ./...
GOWORK=off go build ./...
```

テストはダミーHTTPサーバーを使い、実Zaim APIへは接続しません。GitHub Actionsでもコードの整形・vet・race検査付きテスト・ビルドを検証します。

## 移植元

[zaim-cliの`pkg/zaim`](https://github.com/yone-k/zaim-cli/tree/b94207824e3efd9759d6e8d185a6e660c5b17027/pkg/zaim)から、SDKと既存テストを移植しています。CLI・MCP側の参照変更、互換ラッパー、リリースタグの作成はこの移植には含めていません。

## ライセンス

[MIT](LICENSE)
