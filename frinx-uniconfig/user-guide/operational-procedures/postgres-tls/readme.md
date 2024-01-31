# TLS encryption for Postgres database

By default all the communication to the database is not encrypted.
In deployments where UniConfig is running separately from database, the traffic might be visible to unwanted eyes. Here are the steps to enabling TLS encryption for communication with the database.

## Generating self-signed certificate using OpenSSL

If you already have SSL keys generated, you need to convert them to proper format,
see [Converting SSL keys to proper format](#Converting-SSL-keys-to-proper-format), otherwise you need to generate them.


## Converting SSL keys to proper format

The proper format for the SSL keys is the following:

    Client certificate: PEM encoded X509v3 certificate
    Root certificate: PEM encoded X509v3 certificate
    Key file: PKCS-8 encoded in DER format or PKCS-12 key

The command which needs to be used to convert the keys properly may differ based on the format of the keys in which they are available. They can be converted using OpenSSL version 1.1.1, from command line ```openssl``` command. OpenSSL documentation provides examples for the most common cases. 

To convert to PKCS-8 DER binary format, consult the documentation here: [PKCS-8](https://www.openssl.org/docs/man1.1.1/man1/openssl-pkcs8.html)

To convert to PKCS-12 format, consult the documentation here: [PKCS-12](https://www.openssl.org/docs/man1.1.1/man1/openssl-pkcs12.html)

## Enabling TLS for the database connection

The configuration file that must be modified can
be found on the following path relative to the UniConfig root directory:

```
vim config/application.properties
```

Then edit the **configuration** section in **dbPersistence** section.

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

The TLS related fields are the following:

`db-persistence.connection.enabled-tls` - setting to `true` enables TLS encryption, default is `false`

`db-persistence.connection.tls-client-cert` - specify the **relative** path to the Client certificate from the root UniConfig directory

`db-persistence.connection.tls-client-key` - specify the **relative** path to the Client key from the root UniConfig 
directory, this can be PKCS-12 or PKCS-8 format

`db-persistence.connection.tls-ca-cert` - specify the **relative** path to the root CA certificate from the root UniConfig directory

`db-persistence.connection.ssl-password` - if the `db-persistence.connection.tls-client-key` file is encrypted 
with password, specify it here. It is needed for PKCS-12 keys and for encrypted PKCS-8 keys, this will be ignored for the unencrypted keys.

Do not forget to adjust other database connection parameters accordingly.
