Vercel 外部WebAPIから取得したjsonをそのまま返すサーバーレス関数 (Node.js/TypeScript/Golang/Ruby/Python版)
----

サーバーレス関数の説明は書かない。
https://vercel.com/docs/concepts/functions/serverless-functions

## 対象
- Vercelでhtmlだけの静的サイトを動かしている。
- フロントのJavaScriptからは使えない外部APIを使いたい。
- フロントのフレームワークは使ってない。Next.jsなどならフレームワーク側の機能を使うべき。

## フロントのJavaScriptから使えないAPIは多い
フロントしか知らない人は壮大に勘違いしてる事例がものすごく多いけどWebAPIはフロントから使えるのが当たり前ではない。
クロスオリジンな外部APIはフロントのJSからは使えないのがデフォルトの状態。
CORSによって特別に許可しているだけと認識する。

ここまでが前提として必須な知識。

解決方法は「サーバーサイドで取得すればいい」
サーバーサイドで取得したjsonをそのまま返すapiを「同一オリジン内」もしくは「CORSで許可したクロスオリジン」に用意してフロントからはこのapiを使う。

Vercelのサーバーレス関数なら同一オリジン内にちょっとしたサーバーサイドの処理を用意できる。
（Vercel以外でも全く別の場所に用意してもいいし、S3にjsonファイルを置くだけでもいい。方法は色々ある）

## Node.js
Vercelの設定でNode.js 18.x以上を使うように設定。ネイティブ`fetch()`が18以上でしか使えないので。

`/api/node.js`
```js
const url = 'https://...';

export default async function (req, response) {
    const res = await fetch(url);

    const data = await res.json();

    response.setHeader('Content-Type', 'application/json; charset=utf-8').send(data);
}
```

## TypeScript
- Node.js 18.x以上
- `npm install @vercel/node --save-dev`で`@vercel/node`をインストール。

`/api/typescript.ts`
```typescript
import type {VercelRequest, VercelResponse} from '@vercel/node';

const url: string = 'https://...';

export default async function (req: VercelRequest, response: VercelResponse) {
    const res: Response = await fetch(url);

    const data: string = await res.json();

    response.setHeader('Content-Type', 'application/json; charset=utf-8').send(data);
}
```
TypeScript使う意味はほとんどないのでnode.jsでも同じ。

## Golang
そのまま返すだけならjsonを構造体にする必要もない。

`gopher.go`
```go
package handler

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

const url = "https://..."

func Handler(w http.ResponseWriter, r *http.Request) {
	res, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	defer res.Body.Close()

	json, err := io.ReadAll(res.Body)

	if err != nil {
		log.Fatal(err)
	}

	w.Header().Set("Content-Type", "application/json; charset=utf-8")

	fmt.Fprintln(w, string(json))
}
```

## Ruby

`/api/ruby.rb`
```ruby
require 'net/http'
require 'uri'

Handler = Proc.new do |req, res|
  url = 'https://...'

  res.status = 200
  res['Content-Type'] = 'application/json; charset=utf-8'
  res.body = Net::HTTP.get(URI.parse(url))
end
```

複数の言語でサーバーレス関数を作っていたらファイルサイズのエラーが出た。Ruby単体なら大丈夫だけど色々インストールするような複雑な使い方するとここで引っかかるかも。
```
Error: The Serverless Function "api/ruby" is 202.13mb which exceeds the maximum size limit of 50mb. Learn More: https://vercel.link/serverless-function-size
```

## Python

`/api/python.py`
```python
from http.server import BaseHTTPRequestHandler
import urllib.request

class handler(BaseHTTPRequestHandler):

    def do_GET(self):
        url = 'https://...'

        req = urllib.request.Request(url)
        with urllib.request.urlopen(req) as res:
            body = res.read()

        self.send_response(200)
        self.send_header('Content-type','application/json; charset=utf-8')
        self.end_headers()
        self.wfile.write(body)
        return
```

## 使い方
/api/内にファイルを作ってこのままコピペ。
フロントからはファイル名で決まるURLで使う。

例：`/api/example.js`を作ってコピペしてフロントからは`fetch('/api/example')`で使う。

外部APIはurlで指定。そんなになさそうだけどGET以外で使う仕様なら修正が必要。
APIキーが必要なら環境変数から。

Vercelで公式に対応してる言語なら余計なことは不要で簡単。
