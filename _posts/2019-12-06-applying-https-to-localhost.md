---
layout: post
title: 5분만에 Node.js 로컬호스트에 HTTPS 적용하기 (1)
categories: [HTTPS, Server]
description: applying-https-to-localhost
keywords: HTTPS, Node.js, Server, Client
---

### 들어가는 글,

지난번에 올린 [HTTPS의 기본 개념](https://cydp0127.github.io/2019/12/04/understanding-of-https)에 이어 이번에는 https를 로컬호스트에 적용하는 방법에 대해 알아보겠습니다.

최근 개발하는 서비스에서 중요한 유저 정보 및 카드 정보 등을 위해 https를 개발해야 하는 상황이 생겼습니다. 

보통은 node.js서버에 https를 적용하지만 역시 나 개발을 하려면 로컬 적용이 불가피합니다.

일반적인 웹사이트 또는 서버에서 사용할 https에서는 Certification Authority(CA)가 필요하지만, 본 포스트에서는 임의로 certificate을 발급하는 방법과 trust 방법에 대해 알아보겠습니다.

### Root SSL Certificate

openssl 을 이용하여 우선 rootCA.key 를 생성하여야 합니다. 

아래의 커맨드를 입력하여 생성 가능하며, 생성도중에 passphrase를 물어보게 되는데 추후 key를 사용할때마다 입력되어야 하는 값이므로 잊어버리지 않도록 합니다..

    openssl genrsa -des3 -out rootCA.key 2048

passphrase에 대한 자세한 내용 [Link](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

위에서 생성된 key를 이용하여 이번에는 pem파일을 만들어줍니다. 아래에는 1024일동안 사용가능하며 원하시는 기간만큼 숫자를 조정하시면 됩니다.

    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem

### Certificate Trust 처리

임의로 생성된 certificate이기 때문에 브라우저에서는 믿을 수 없다고 처리됩니다. 따라서 호스트역할을 하는 (여기서는 Mac)서버에서 trust 처리를 해주어야 합니다.

mac의 keychain access에서 방금 생성한 certificate을 찾아 always trust 처리를 해줍니다.

keychain access

![](/images/posts/https/keychain.png)

여기에 생성한 certificate을 import 해준다면 위와 같이 등록이 되며, 더블클릭 후 trust에서 always trust로 처리 해주면 됩니다.

자 이제 거의 다 왔습니다.

사실 이 상태로 certificate을 서버에 적용하여 돌려보시면 HTTPS로 동작은 합니다만 여전히 브라우저에서는 Not Secure라고 표시됩니다.

![](/images/posts/https/not-secure.png)

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
    ST=StateName
    L=CityName
    O=OrganizationName
    OU=OrganizationUnitName
    emailAddress=helloworld@gililab.com
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

이렇게 certificate 생성 부분이 끝났습니다.

임의로 생성한 certificate은 안전하지 않으며 절대 실제 production에서는 사용하지 않으시길 바랍니다.

생성된 키를 Node.js 서버에 적용시키는 방법은 다음 포스트에서 다루도록 하겠습니다.

### Reference

[https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec](https://www.freecodecamp.org/news/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec/)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server](https://loopback.io/doc/en/lb2/Preparing-for-deployment.html#create-the-https-server)

[https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file](https://support.comodo.com/index.php?/Knowledgebase/Article/View/364/17/what-is-a-passphrase-and-how-can-i-change-the-passphrase-on-my-private-key-file)

[https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl](https://stackoverflow.com/questions/47957538/preparing-loopback-to-use-ssl)
