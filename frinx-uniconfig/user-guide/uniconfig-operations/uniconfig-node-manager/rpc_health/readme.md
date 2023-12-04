# RPC health

This RPC checks if UniConfig is running. If database persistence is enabled, it
also checks the database connection.

## RPC examples

RPC input is empty. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:health' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

**Response if database persistence is disabled:**

```json RPC Response, Status: 200
{
  "output": {
    "healthy": true,
    "message": "DB persistence is disabled"
  }
}
```

**Response if database persistence is enabled and the database connection is valid:**

```json RPC Response, Status: 200
{
  "output": {
    "healthy": true,
    "message": "DB connection is alive"
  }
}
```

**Response if database persistence is enabled and database connection is not valid:**

```json RPC Response, Status: 200
{
  "output": {
    "healthy": false,
    "message": "Error connecting to DB"
  }
}
```
