HTTP:
```java
    case HTTP -> {
                    server = HttpServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), 0), 0);
                    server.createContext("/" + account, new AzureHttpHandler(account, container, actualAuthHeaderPredicate));
                    server.start();

                    oauthTokenServiceServer = HttpServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), 0), 0);
                    oauthTokenServiceServer.createContext(
                        "/",
                        new AzureOAuthTokenServiceHttpHandler(workloadIdentityBearerToken, federatedToken, tenantId, clientId)
                    );
                    oauthTokenServiceServer.start();
                }
```
메인 HTTP 서버 생성:
루프백 주소와 동적 포트(0)로 HttpServer를 생성합니다.
account, container, actualAuthHeaderPredicate를 파라미터로 받는 AzureHttpHandler를 통해 Azure 관련 요청을 처리하는 컨텍스트("/" + account)를 설정합니다.
서버를 시작합니다.
OAuth 토큰 서비스 서버 생성:
별도의 HttpServer 인스턴스를 루프백 주소와 동적 포트로 생성합니다.
AzureOAuthTokenServiceHttpHandler를 사용하여 Azure OAuth 인증이나 토큰 관리를 담당하는 루트 컨텍스트("/")를 설정합니다.
이 OAuth 서버를 시작합니다.

  
HTTPS:
  ```java
  case HTTPS -> {
                    final var tmpdir = ESTestCase.createTempDir();
                    final var certificates = PemUtils.readCertificates(List.of(copyResource(tmpdir, "azure-http-fixture.pem")));
                    assertThat(certificates, hasSize(1));
                    final SSLContext sslContext = SSLContext.getInstance("TLS");
                    sslContext.init(
                        new KeyManager[] {
                            KeyStoreUtil.createKeyManager(
                                new Certificate[] { certificates.get(0) },
                                PemUtils.readPrivateKey(copyResource(tmpdir, "azure-http-fixture.key"), () -> null),
                                null
                            ) },
                        null,
                        new SecureRandom()
                    );
                    {
                        final var httpsServer = HttpsServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), 0), 0);
                        this.server = httpsServer;
                        httpsServer.setHttpsConfigurator(new HttpsConfigurator(sslContext));
                        httpsServer.createContext("/" + account, new AzureHttpHandler(account, container, actualAuthHeaderPredicate));
                        httpsServer.start();
                    }
                    {
                        final var httpsServer = HttpsServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), 0), 0);
                        this.oauthTokenServiceServer = httpsServer;
                        httpsServer.setHttpsConfigurator(new HttpsConfigurator(sslContext));
                        httpsServer.createContext(
                            "/",
                            new AzureOAuthTokenServiceHttpHandler(workloadIdentityBearerToken, federatedToken, tenantId, clientId)
                        );
                        httpsServer.start();
                    }
                }
```

SSL 컨텍스트 설정:
임시 디렉터리(tmpdir)를 생성하고 SSL 인증서를 해당 디렉터리에서 읽어옵니다.
PemUtils.readCertificates를 통해 인증서를 로드합니다.
TLS로 설정된 SSLContext 인스턴스를 생성하고, 인증서와 개인 키를 관리하는 KeyManager로 초기화합니다.
메인 HTTPS 서버 생성:
루프백 주소와 동적 포트를 가진 HttpsServer를 설정합니다.
HttpsConfigurator를 통해 SSL 설정을 적용하고, 준비된 SSLContext를 연결하여 보안 통신을 설정합니다.
HTTP와 동일하게 "/" + account 컨텍스트를 설정하되, 이번에는 HTTPS 서버 인스턴스를 사용합니다.
HTTPS 서버를 시작합니다.
OAuth 토큰 서비스 HTTPS 서버 생성:
HTTPS를 통해 OAuth 토큰을 처리할 별도의 HttpsServer를 설정하여 SSL로 보안합니다.
루트 컨텍스트 "/"에 동일한 AzureOAuthTokenServiceHttpHandler를 설정합니다.
이 OAuth HTTPS 서버를 시작합니다.
각 케이스는 protocol에 맞는 서버를 설정하여, Azure 관련 HTTP와 HTTPS 요청을 다룰 수 있도록 합니다.


                  
