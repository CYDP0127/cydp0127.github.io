---
layout: post
title: 5분안에 Node.js 로컬호스트에 HTTPS 적용하기 (2)
categories: [HTTPS, Server]
description: applying-https-to-localhost
keywords: HTTPS, Node.js, Server, Client
---

지난번 [포스트](https://cydp0127.github.io/2019/12/06/applying-https-to-localhost-1)에서 임의로 certificate을 생성하는 방법을 알아 보았는데요, 이번에는 생성된 certificate을 서버에 적용 시키는 방법에 대해 알아보겠습니다.

### 생성된 certificate node.js 서버에 적용하기

지난번 포스트에서 생성한 server.key와 server.crt파일을 프로젝트 폴더에 위치 시킵니다.

저는 프로젝트 폴더 하위에 private 이라는 폴더를 생성하여 server.key와 server.crt파일을 옮겨 넣었습니다.

이제 ssl 데이터를 가지고 있는 config 파일을 생성하여 줍니다.

ssl-config.js

``` javascript
const path = require('path');
const fs = require('fs');
    
exports.privateKey = fs.readFileSync(path.join(__dirname, '../private/server.key')).toString();
exports.certificate = fs.readFileSync(path.join(__dirname, '../private/server.crt')).toString();
```

시스템마다 다르겠지만 서버의 호스팅 옵션값을 가지고 있는 파일에 아래와 같이 적용합니다.

loopback framework을 사용하는 저 같은 경우는 config.local.js 파일에 적용하였습니다.
``` javascript
{
  restApiRoot: '/api',
  host: process.env.HOST || '0.0.0.0',
  port: process.env.PORT || 80,
  url: 'https://localhost',
}
```
마지막으로 server.js에 https를 적용시켜줍니다.!
``` javascript
...
const https = require('https');
const sslConfig = require('../config/ssl-config');
...

const options = {
  key: sslConfig.privateKey,
  cert: sslConfig.certificate,
  passphrase: 'abcd' // certificate을 생성하면서 입력하였던 passphrase 값
};
...
const app = module.exports;
app.start = function () {
  // start the web server
  const server = https.createServer(options, app);
  return server.listen(app.get('port'), () => {
    app.emit('started');
    const baseUrl = app.get('url') + ':' + app.get('port');
    console.info('Web server listening at: %s', baseUrl);
    if (app.get('loopback-component-explorer')) {
      const explorerPath = app.get('loopback-component-explorer').mountPath;
      console.info('Browse your REST API at %s%s', baseUrl, explorerPath);
    }
  });
};
```
![](/images/posts/https/secure.png)

정상적으로 적용 되었습니다!

### 마치는 글

이렇게 해서 https certificate 생성 부터 Node.js에 적용하는 방법까지 알아보았습니다.

certificate을 생성하는 과정은 다소 복잡해 보일 수 있으나 서버에 적용시키는 방법은 생각보다 많이 간단합니다.

질문이나 피드백이 있다면 언제든지 알려주세요 !

### Reference

[https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec](https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec/)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server](https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl](https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl)
