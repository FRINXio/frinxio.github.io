# TLS encryption for Postgres database

By default, communications with the database are unencrypted. In deployments
where UniConfig is running separately from the database, network traffic may be
visible to outside parties.

The sections below describe how to enable TLS encryption for communications with
the database.

## Generate self-signed certificate using OpenSSL

If you have already generated your SSL keys, you must convert them to the
correct format. See [Convert SSL keys to correct format](#Convert-SSL-keys-to-correct-format)

If not, you must first generate your keys.

## Convert SSL keys to correct format

The correct format for SSL keys is as follows:

    Client certificate: PEM encoded X509v3 certificate
    Root certificate: PEM encoded X509v3 certificate
    Key file: PKCS-8 encoded in DER format or PKCS-12 key

The command for converting the keys may differ based on the format of your
existing keys. They can be converted using OpenSSL version 1.1.1 with the
```openssl``` command.

The OpenSSL documentation provides examples for most
common cases:
- To convert to PKCS-8 DER binary format, see 
[PKCS-8](https://www.openssl.org/docs/man1.1.1/man1/openssl-pkcs8.html).
- To convert to PKCS-12 format, see 
[PKCS-12](https://www.openssl.org/docs/man1.1.1/man1/openssl-pkcs12.html).

## Enable TLS for database connections

Edit the configuration file at the following path relative to the UniConfig
root directory: 

```
vim config/application.properties
```

Modify the `connection` section within the `dbPersistence` section:

Example:

```properties
# DB persistence settings
db-persistence.embedded-database.enabled=true
db-persistence.embedded-database.data-dir=./data/pg_dir
db-persistence.embedded-database.clean-data-dir=true

db-persistence.connection.db-name=uniconfig
db-persistence.connection.username=uniremote
db-persistence.connection.password=unipass
db-persistence.connection.uri=jdbc:postgresql://
db-persistence.connection.driver-class-name=org.postgresql.Driver
db-persistence.connection.connection-timeout=30000
db-persistence.connection.max-lifetime=1800000
db-persistence.connection.min-idle-connections=10
db-persistence.connection.max-db-pool-size=20
db-persistence.connection.socket-read-timeout=20
db-persistence.connection.enabled-tls=false
db-persistence.connection.tls-client-cert=./client.pks
db-persistence.connection.tls-client-key=./client.key
db-persistence.connection.tls-ca-cert=./ca.pks
db-persistence.connection.ssl-password=
db-persistence.connection.database-locations[0].host=127.0.0.1
db-persistence.connection.database-locations[0].port=26257
db-persistence.connection.repair-schema-history=false
#db-persistence.uniconfig-instance.instance-name=
db-persistence.uniconfig-instance.host=127.0.0.1
```

TLS-related fields include the following:

`db-persistence.connection.enabled-tls` - Set to `true` to enable TLS
encryption. The default value is `false`.

`db-persistence.connection.tls-client-cert` - Specify the *relative* path from
the root UniConfig directory to the Client certificate.

`db-persistence.connection.tls-client-key` - Specify the *relative* path from
the root UniConfig directory to the Client key. Can be PKCS-12 or PKCS-8 format.

`db-persistence.connection.tls-ca-cert` - Specify the *relative* path  from the
root UniConfig directory to the root CA certificate.

`db-persistence.connection.ssl-password` - If the file specified in
`db-persistence.connection.tls-client-key` is encrypted with a password, specify
the password here. Required for PKCS-12 keys and for encrypted PKCS-8 keys.
Ignored for unencrypted keys.

!!!
Do not forget to adjust other database connection parameters accordingly.
!!!