# TLS-based Authentication

TLS authentication is disabled in the default version of UniConfig.

To enable TLS for RESTCONF, perform the following two steps:

1. Set up key-store and trust-store to hold all keys and certificates. If
    authentication of individual clients is not required, trust-store is not
    required. Key-store must always be initialized.
2. Enable TLS in UniConfig via the **application.properties** configuration file.

## Set key-store and trust-store

To prepare key-store and trust-store:

1. In the UniConfig root directory, create a new directory that will contain
    key-store and, optionally, trust-store files. For example:

```
mkdir ./tls
cd ./tls
```

2.  Create a new key-store. There are two options depending on whether you
    already own the certificate that you want to use for identification of
    UniConfig on the RESTCONF layer:

    a.  Create a new key-store with the generated RSA key-pair (the example below
     uses a length of 2048 and validity of 365 days). After executing the
     following command, the prompt will ask you for information about the
     currently generated certificate that is pushed into the newly generated
     key-store secured by a password (this secret will be used later in the
     configuration file - make sure that you remember it).

       ```
       keytool -keystore .keystore -alias jetty -genkey -keyalg RSA -storetype PKCS12 -validity 365 -keysize 2048
       ```

    b.  Create a new key-store with the already-generated RSA key-pair (your
     certificate that you want to use for authentication in ODL).

     ```
     keytool -import -file [your-certificate-file] -alias jetty -keystore .keystore
     ```

3. (Optional step) Create a new trust-store using an existing certificate (an
    empty trust-store cannot be created). If you have multiple client
    certificates, they can be pushed to trust-store by executing the same command
    multiple times (the alias must be unique for each of the imported
    certificates). For example:

```
keytool -import -file [client-app-certificate] -alias [unique-name-of-certificate] -keystore .truststore
```

!!!
You can easily convert OPENSSL PEM certificates to the DER format supported
by keytool:

```
openssl x509 -outform der -in certificate.pem -out certificate.der
```
!!!

!!!

If your application must own the distribution's certificate, you can export the
certificate from the generated key-pair that we have pushed into the keystore
(PKCS12 or OPENSSL format):

```
keytool -export -keystore .keystore -alias jetty -file odl.cer
openssl pkcs12 -in .keystore -out certificate.pem
```
!!!

## Enable TLS in UniConfig

After preparing key-store and trust-store, you need to point UniConfig to these
storages and explicitly enable TLS via a flag.

Modify the following configuration file at a path relative to the UniConfig root
directory:

```
vim config/application.properties
```

Next, edit TLS configuration section, uncommenting and editing the necessary properties.

The example snippet below enables TLS authentication, disables user-based
authentication (hence trust-store is not required) and points UniConfig to the
key-store file created in the previous section:

```properties
# TLS settings
server.http2.enabled=true
server.ssl.enabled=true
server.ssl.protocol=TLS
server.ssl.client-auth=none
server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
#server.ssl.ciphers=
#server.ssl.trust-store=tls/.truststore
#server.ssl.trust-store-password=trust-pass
server.ssl.key-store=tls/.keystore
server.ssl.key-store-password=passtest
# Enable SNI
#server.ssl.sni-enabled=true
```

If your deployment requires authentication for individual RESTCONF users,
truststore is **required** and you need to set 
`server.ssl.client-auth=need`

```properties
# TLS settings
server.http2.enabled=true
server.ssl.enabled=true
server.ssl.protocol=TLS
server.ssl.client-auth=need
#server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
#server.ssl.ciphers=
server.ssl.trust-store=tls/.truststore
server.ssl.trust-store-password=trust-pass
server.ssl.key-store=tls/.keystore
server.ssl.key-store-password=passtest
# Enable SNI
#server.ssl.sni-enabled=true
```

JVM provides secure defaults, but you can also specify included cipher suites and TLS version,
overriding default settings.
Example configuration
that includes support for TLS 1.2 and TLS 1.3 with some of the most common and strongest ciphers available:

```properties
server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
server.ssl.ciphers=TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

SNI (Server Name Indication) is disabled by default.
To enable it, uncomment the line as shown below:

```properties
# Enable SNI
server.ssl.sni-enabled=true
```