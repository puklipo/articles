# Vercel Serverless Function to Return JSON from an External Web API (Node.js/TypeScript/Golang/Ruby/Python versions)

---

This article does not explain serverless functions themselves.

https://vercel.com/docs/concepts/functions/serverless-functions

## Target Audience
- You are running a static HTML-only site on Vercel.
- You want to use an external API that cannot be used from front-end JavaScript.
- You are not using a front-end framework. If you are using Next.js or similar, you should use the framework's features.

## Many APIs Cannot Be Used From Front-end JavaScript
There are many instances where people who only know front-end development have a major misunderstanding: it's not a given that Web APIs can be used from the front-end.
By default, cross-origin external APIs cannot be used from front-end JS.
Recognize that they are only specially permitted by CORS.

This is essential prerequisite knowledge.

The solution is "fetch it on the server-side."
Prepare an API on the "same origin" or "CORS-allowed cross-origin" that returns the JSON fetched on the server-side, and use this API from the front-end.

With Vercel serverless functions, you can prepare a small amount of server-side processing within the same origin.
(You can also prepare it in a completely different location other than Vercel, or even just place a JSON file on S3. There are various methods.)

## Node.js
Configure Vercel to use Node.js 18.x or higher, as native `fetch()` is only available in version 18 and above.

`/api/node.js`
```javascript
const url = 'https://...';

export default async function (req, response) {
    const res = await fetch(url);

    const data = await res.json();

    response.setHeader('Content-Type', 'application/json; charset=utf-8').send(data);
}
```

## TypeScript
- Node.js 18.x or higher
- Install `@vercel/node` with `npm install @vercel/node --save-dev`.

`/api/typescript.ts`
```typescript
import type {VercelRequest, VercelResponse} from '@vercel/node';

const url: string = 'https://...';

export default async function (req: VercelRequest, response: VercelResponse) {
    const res: Response = await fetch(url);

    const data: any = await res.json(); // Changed 'string' to 'any' for broader compatibility

    response.setHeader('Content-Type', 'application/json; charset=utf-8').send(data);
}
```
There's little point in using TypeScript here, so it's almost the same as Node.js.

## Golang
If you're just returning it as is, there's no need to convert the JSON to a struct.

`gopher.go` (assuming the filename corresponds to the API endpoint, e.g., `/api/gopher`)
```go
package handler // The package name must be 'handler' for Vercel

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

const url = "https://..."

func Handler(w http.ResponseWriter, r *http.Request) { // The function must be named Handler
	res, err := http.Get(url)
	if err != nil {
		log.Fatal(err) // Consider http.Error for better error handling in production
	}
	defer res.Body.Close()

	jsonBody, err := io.ReadAll(res.Body) // Renamed 'json' to 'jsonBody' to avoid conflict

	if err != nil {
		log.Fatal(err) // Consider http.Error
	}

	w.Header().Set("Content-Type", "application/json; charset=utf-8")

	fmt.Fprintln(w, string(jsonBody))
}
```

## Ruby

`/api/ruby.rb`
```ruby
require 'net/http'
require 'uri'
require 'json' # Required for parsing if the external API returns non-string JSON

Handler = Proc.new do |req, res|
  api_url = 'https://...' # Renamed 'url' to 'api_url' to avoid conflict with Vercel's 'url' property on req

  uri = URI.parse(api_url)
  response_body = Net::HTTP.get(uri)

  res.status = 200
  res['Content-Type'] = 'application/json; charset=utf-8'
  res.body = response_body # Assuming the API returns a JSON string
end
```

If you were creating serverless functions in multiple languages, you might encounter a file size error. Ruby alone should be fine, but complex usage involving many installations might hit this limit.
```
Error: The Serverless Function "api/ruby" is 202.13mb which exceeds the maximum size limit of 50mb. Learn More: https://vercel.link/serverless-function-size
```

## Python

`/api/python.py`
```python
from http.server import BaseHTTPRequestHandler
import urllib.request
import json # For potential future use if you need to manipulate the JSON

class handler(BaseHTTPRequestHandler):

    def do_GET(self):
        api_url = 'https://...' # Renamed 'url'

        req = urllib.request.Request(api_url)
        with urllib.request.urlopen(req) as res:
            body = res.read()

        self.send_response(200)
        self.send_header('Content-type','application/json; charset=utf-8')
        self.end_headers()
        self.wfile.write(body)
        return
```

## How to Use
Create a file in the `/api/` directory and copy-paste the code.
Use it from the front-end with the URL determined by the filename.

Example: Create `/api/example.js`, copy-paste the code, and use it from the front-end with `fetch('/api/example')`.

Specify the external API in the `url` variable. If the API specification uses methods other than GET (though unlikely), modifications will be necessary.
If an API key is required, get it from environment variables.

It's easy with languages officially supported by Vercel, with no extra hassle.
