
#SSL与基本接入认证


Marathon可以通过SSL进行API端点的安全访问也恶意通过HTTP基本接入认证进行有限访问。如果计划使用基本认证方式，我们建议使用SSL方式，因为其他方式由于用户名和密码未被加密，信息容易被其他方获取。

##Enabling SSL##使用SSL


如果已经有了Java keystore，可以将其连通密码一起通过SSL传递至Marathon
    
    $ cd /path/to/marathon
    $ ./bin/start --master zk://localhost:2181/mesos \
      --zk zk://localhost:2181/marathon \
       --ssl_keystore_path marathon.jks \
       --ssl_keystore_password $MARATHON_JKS_PASSWORD



在默认情况下，Marathon使用8443端口来服务SSL（可以通过`--https_port`命令来更改端口），一旦Marathon运行，将会通过HTTPS端口来接入API以及UI:

    $ curl https://localhost:8443/v2/apps



###生成带有SSL密钥的keystore并进行认证


> 注意：现在的浏览器以及大部分工具，将会对于Marathon API以及UI的访问进行告警以及于除非SSL已经在keystore认证并且被认证策略所信任。否则只能从已经被信任的方面购买SSL认真或者将您公司的root认证给Marathon API以及UI的使用者。




1.通过[OpenSSL](https://www.openssl.org/docs/apps/genrsa.html)生成RSA私有密钥

    # Set key password to env variable `MARATHON_KEY_PASSWORD`
    $ openssl genrsa -des3 -out marathon.key -passout "env:MARATHON_KEY_PASSWORD"






2.通过下面的方法为密钥获得认证

- （建议方式）从被信任的认证权威中购买认证。这个将确保Marathon实例的API以及UI信任SSL认证，为用户减少额外的步奏


- （不安全方式）为密钥生成一个认证。这个命令会向与keystore交互的信息进行提示。“Common Name”必须是要使用认证的机器的完全主机名。
    
        
    	# Read key password from env env variable `MARATHON_KEY_PASSWORD`
        $ openssl req -new -x509 -key marathon.key \
      	-passin "env:MARATHON_KEY_PASSWORD" \
     	-out self-signed-marathon.pem





3.将密钥和认证文件合并至PKCS12格式的文件中，PKCS12这种格式也是Java keystore用的格式。如果接受的认证文件不是.pem格式，需要参考[Jetty SSL configuration](http://www.eclipse.org/jetty/documentation/current/configuring-ssl.html#loading-keys-and-certificates)文档将它转换为.pem。

    # Read key password from env variable `MARATHON_KEY_PASSWORD`
    # Set PKCS password to env variable `MARATHON_PKCS_PASSWORD`
    $ openssl pkcs12 -inkey marathon.key \
     -passin "env:MARATHON_KEY_PASSWORD" \
      -name marathon \
     -in trusted.pem \
      -password "env:MARATHON_PKCS_PASSWORD" \
     -chain -CAFile "trustedCA.crt" \
       -export -out marathon.pkcs12




4.导入keystore

    # Read PKCS password from env variable `MARATHON_PKCS_PASSWORD`
    # Set JKS password to env variable `MARATHON_JKS_PASSWORD`
    $ keytool -importkeystore -srckeystore marathon.pkcs12 \
     -srcalias marathon \
     -srcstorepass $MARATHON_PKCS_PASSWORD \
     -srcstoretype PKCS12 \
     -destkeystore marathon.jks \
     -deststorepass $MARATHON_JKS_PASSWORD




5.通过创建的keystore以及其密码启动Marathon

    # Read JKS password from env variable `MARATHON_JKS_PASSWORD`
    $ ./bin/start --master zk://localhost:2181/mesos \
      --zk zk://localhost:2181/marathon \
       --ssl_keystore_path marathon.jks \
       --ssl_keystore_password $MARATHON_JKS_PASSWORD




6.通过HTTP端口接入Marathon API与UT（默认端口8443）

    https://<MARATHON_HOST>:8443


###使用基本接入认证


> 备注：虽然可以使用基本认证，但是我们强烈建议使用SSL。如果SSL不可用，Marathon实例的用户名和密码将会使用无加密形式进行传输，可能会被其他方获取到相关信息。


将用户名和密码使用"："进行隔离传递至`--http_credentials `命令将会开启基本认证
备注：用户名不能包含":".

    $ cd /path/to/marathon
    $ ./bin/start --master zk://localhost:2181/mesos \
      --zk zk://localhost:2181/marathon \
	  --http_credentials "cptPicard:topSecretPa$$word" \
       --ssl_keystore_path /path/to/marathon.jks \
       --ssl_keystore_password $MARATHON_JKS_PASSWORD

