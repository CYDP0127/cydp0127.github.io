---
layout: post
title: Node.js 로컬호스트에 HTTPS 적용하기
categories: [HTTPS, Node.js, Server]
description: applying-https-to-localhost
keywords: HTTPS, Node.js, Server
---

### 들어가는 글,

최근 개발하는 서비스에서 중요한 유저 정보 및 카드 정보 등을 위해 https를 개발해야 하는 상황이 생겼습니다.

보통은 node.js서버에 https를 적용하지만 역시나 개발을 하려면 로컬 적용이 불가피합니다.

일반적인 웹사이트 또는 서버에서 사용할 https에서는 Certification Authority(CA)가 필요하지만, 본 포스트에서는 임의로 certificate을 발급하는 방법과 trust 방법, Node.js 서버에 적용 방법에 대해 다뤄 보겠습니다.

### Root SSL Certificate

openssl 을 이용하여 우선 rootCA.key 를 생성하여야 합니다.

아래의 커맨드를 입력하여 생성 가능하며, 생성도중에 passphrase를 물어보게 되는데 추후 key를 사용할때마다 입력되어야 하는 값이므로 잊어버리지 않도록 합니다..

    openssl genrsa -des3 -out rootCA.key 2048

passphrase에 대한 자세한 내용 (여기)를 참고[[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)]

위에서 생성된 key를 이용하여 이번에는 pem파일을 만들어줍니다. 아래에는 1024일동안 사용가능하며 원하시는 기간만큼 숫자를 조정하시면 됩니다.

    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem

### Certificate Trust 처리

임의로 생성된 certificate이기 때문에 브라우저에서는 믿을 수 없다고 처리됩니다. 따라서 호스트역할을 하는 (여기서는 Mac)서버에서 trust 처리를 해주어야 합니다.

mac의 keychain access에서 방금 생성한 certificate을 찾아 always trust 처리를 해줍니다.

keychain access

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/287dea6d-36cc-4aea-8e31-84df691069e3/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/287dea6d-36cc-4aea-8e31-84df691069e3/Untitled.png)

여기서 생성한 certificate을 import 해준다면 위와같이 등록이 되며, 더블클릭 후 trust에서 always trust로 처리해주면 됩니다.

자 이제 거의 다 왔습니다.

사실 이 상태로 certificate을 서버에 적용하여 돌려보시면 HTTPS로 동작은 합니다만 여전히 브라우저에서는 Not Secure라고 표시됩니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22247416-b47e-4c96-9bd2-606c85de79ea/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22247416-b47e-4c96-9bd2-606c85de79ea/Untitled.png)

### 로컬에서 사용할 수 있도록 domain 설정하기

방금전 certificate을 생성하였던 디렉토리에서 아래와 같은 파일을 생성하여 줍니다.

cnf파일은 키 생성시 입력하였던 옵션값을 파일로 만들어 매번 생성 시 일일이 값을 입력하지 않도록 해줍니다.

server.csr.cnf

    [req]
    default_bits = 2048
    prompt = no
    default_md = sha256
    distinguished_name = dn

    [dn]
    C=US
    ST=RandomState
    L=RandomCity
    O=RandomOrganization
    OU=RandomOrganizationUnit
    emailAddress=hello@example.com
    CN = localhost

v3.ext

    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = localhost

생성이 완료되었다면 아래의 커맨드로 server.key와 crt파일을 만들어줍니다.

    openssl req -new -sha256 -nodes -out server.csr -newkey rsa:2048 -keyout server.key -config <( cat server.csr.cnf )

    openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 500 -sha256 -extfile v3.ext

### 생성된 certificate node.js 서버에 적용하기

드디어 생성부분이 끝났는데요, 서버에 적용하는 방법은 굉장히 간단합니다.

위에서 생성된 server.key와 server.crt파일을 서버 프로젝트 디렉토리에 위치 시킵니다. 저같은 경우는 프로젝트 폴더 하위에 private 이라는 폴더를 생성하여 server.key와 server.crt파일을 옮겨 넣었습니다.

이제 ssl 데이터를 가지고 있는 config 파일을 생성하여 줍니다.

ssl-config.js

    const path = require('path');
    const fs = require('fs');

    exports.privateKey = fs.readFileSync(path.join(__dirname, '../private/server.key')).toString();
    exports.certificate = fs.readFileSync(path.join(__dirname, '../private/server.crt')).toString();

시스템마다 다르겠지만 서버의 호스팅 옵션값을 가지고 있는 파일에 아래와 같이 적용합니다.

loopback을 사용하는 저같은 경우는 config.local.js 파일에 적용하였습니다.

    {
      restApiRoot: '/api',
      host: process.env.HOST || '0.0.0.0',
      port: process.env.PORT || 80,
      url: 'https://localhost',
    }

마지막으로 server.js에 https를 적용시켜줍니다.!

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

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc95a6d2-a594-4bb9-9bb5-1f891828cce8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc95a6d2-a594-4bb9-9bb5-1f891828cce8/Untitled.png)

정상적으로 적용 되었습니다!

### 마치는 글

이렇게해서 https certificate 생성부터 Node.js에 적용하는 방법까지 알아보았습니다.

위와 같은 방법은 로컬에서 테스트용으로만 사용하여야 하며 절대 production에는 적용하지 마시길 바랍니다.

### Reference

[https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec](https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec/)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server](https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl](https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl)
