# TLS-based Authentication

TLS authentication is disabled in the default version of UniConfig.

To enable TLS for RESTCONF, perform the following two steps:

1. Set up key-store and trust-store to hold all keys and certificates. If
    authentication of individual clients is not required, trust-store is not
    required. Key-store must always be initialized.
2. Enable TLS in UniConfig via the **lighty** configuration file.

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
penssl pkcs12 -in .keystore -out certificate.pem
```
!!!

## Enable TLS in UniConfig

After preparing key-store and trust-store, you need to point UniConfig to these
storages and explicitly enable TLS via a flag.

Modify the following configuration file at a path relative to the UniConfig root
directory:

```
vim config/lighty-uniconfig-config.json
```

Next, append a TLS configuration snippet to the configuration file (the snippet
must be placed under the root JSON node).

The example snippet below enables TLS authentication, disables user-based
authentication (hence trust-store is not required) and points UniConfig to the
key-store file created in the previous section:

```json
{
  "tls": {
    "enabledTls": true,
    "enabledClientAuthentication": false,
    "keystorePath": "tls/.keystore",
    "keystorePassword": "key-pass"
  }
}
```

If your deployment requires authentication for individual RESTCONF users, you
should also specify the trust-store fields by setting
`enabledClientAuthentication` to `true`.

```json
{
  "tls": {
    "enabledTls": true,
    "enabledClientAuthentication": true,
    "keystorePath": "tls/.keystore",
    "keystorePassword": "key-pass",
    "truststorePath": "tls/.truststore",
    "truststorePassword": "trust-pass"
  }
}
```

You can also specify included or excluded cipher suites and TLS versions, which
then can or cannot be used to establish a secured tunnel between the Jetty
server and clients.

The following default configuration is based on actual recommendations (adjust
as needed):

```json
{
  "tls": {
    "includedProtocols": [
      "TLSv1.2",
      "TLSv1.3"
    ],
    "excludedProtocols": [
      "TLSv1",
      "TLSv1.1",
      "SSL",
      "SSLv2",
      "SSLv2Hello",
      "SSLv3"
    ],
    "includedCipherSuites": [
      "TLS_ECDHE.*",
      "TLS_DHE_RSA.*"
    ],
    "excludedCipherSuites": [
      ".*MD5.*",
      ".*RC4.*",
      ".*DSS.*",
      ".*NULL.*",
      ".*DES.*"
    ]
  }
}
```

!!!
It is sufficient to specify only included protocols and cipher suites (so that
all other entries are denied), or excluded protocols and cipher suites (so that
all other entries are permitted).

If you specify the same entries under both included and excluded cipher suites
or protocols, the excluded entry is given higher priority. For example, the
final set of usable cipher suites is: setOf(includedCipherSuites),
setOf(excludedCipherSuites).
!!!