# Device discovery

The **device-discovery RPC** is used to verify reachable devices in a network.
You can either check a single IP address in IPv4 format, a network or a range of
addresses. Additionally, you can specify a port or range of ports (TCP or UDP)
that are checked if they are open. The ICMP protocol is used to check the
availability of devices. 

RPC input consists of a list of all IP addresses to check (IPv4 or IPv6, a
single IP address or a network with a prefix, or a range of IP addresses).
Additionally, it contains the TCP/UDP ports that should be checked if they are
open or not on the given addresses.

RPC output shows if the IP addresses are reachable via the ICMP protocol. For
every IP address, a list of open TCP/UPD ports is also included.

For testing, you need to add your IP address to the configuration JSON file.
The configuration file is located at:

**\~/FRINX-machine/config/uniconfig/frinx/uniconfig/config/application.properties**

When running UniConfig stand-alone, the config file is in the **config** folder:

**/opt/uniconfig-frinx/config/application.properties**

Execute the `ifconfig` command in the terminal and look for an interface. If you
are using a VPN, the interface is often called `tun0`. If not, look for a
different interface. From the interface, copy the `inet` address and paste it
into the file.

```properties Snippet
device-discovery.local-address=
device-discovery.initial-pool-size=1
device-discovery.max-pool-size=20
device-discover-keepalive-time=60
device-discovery.address-check-limit=254
```

The previous snippet contains the following additional parameters:

-   `initial-pool-size` - Thread pool used by the executor.
-   `max-pool-size` - Size of the executor. If the request contains a large
    number of addresses, consider raising the value.
-   `keepalive-time` - Time before the execution of a specified task is timed
    out (in seconds).
-   `addressCheckLimit` - Maximum number of addresses checked. If a request
    contains more addresses, it will not be successful.

To discover hosts and ports in listening state in a network, do not add the
network and broadcast address of that network. For example, to check the network
`192.168.1.0/24`, use one of the following:

-   `"network": "192.168.1.0/24"`
-   `"start-ipv4-address": "192.168.1.1", "end-ipv4-address":
    "192.168.1.254"`

If you specify the range using a network statement, the network address and
broadcast address are not included in the discovery process. If you specify the
range via range statements, make sure that only hosts addresses are included in
the specified range.

## RPC examples

### Successful example

RPC input contains a network with the `/29` prefix. Addresses in the network and
desired ports are checked for availability.

RPC output contains reachable addresses in the network and all open TCP/UDP
ports.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw ' {
     "input": {
         "address": [
             {
                 "network": "192.168.1.0/29"
             }
         ],
         "tcp-port": [
             {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "device": [
      {
        "host": "192.168.1.1",
        "available-udp-ports": [
          50,
          51,
          52,
          69
        ],
        "unavailable-tcp-ports": [
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88
        ],
        "available-tcp-ports": [
          80,
          443
        ],
        "is-host-reachable": true
      },
      {
        "host": "192.168.1.2",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.3",
        "unavailable-tcp-ports": [
          80,
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88,
          443
        ],
        "is-host-reachable": true,
        "unavailable-udp-ports": [
          50,
          51,
          52,
          53,
          69
        ]
      },
      {
        "host": "192.168.1.4",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.5",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.6",
        "unavailable-tcp-ports": [
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88
        ],
        "available-tcp-ports": [
          80,
          443
        ],
        "is-host-reachable": true,
        "unavailable-udp-ports": [
          50,
          51,
          52,
          53,
          69
        ]
      }
    ]
  }
}
```

### Successful example

RPC input contains a range of addresses. Addresses and desired ports are checked
for availability.

RPC output contains reachable addresses and all open TCP/UDP ports.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "address": [
            {
                "start-ipv4-address": "192.168.1.1",
                "end-ipv4-address": "192.168.1.8"
            }
        ],
        "tcp-port": [
            {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "device": [
      {
        "host": "192.168.1.1",
        "available-udp-ports": [
          50,
          51,
          52,
          69
        ],
        "unavailable-tcp-ports": [
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88
        ],
        "available-tcp-ports": [
          80,
          443
        ],
        "is-host-reachable": true
      },
      {
        "host": "192.168.1.2",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.3",
        "unavailable-tcp-ports": [
          80,
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88,
          443
        ],
        "is-host-reachable": true,
        "unavailable-udp-ports": [
          50,
          51,
          52,
          53,
          69
        ]
      },
      {
        "host": "192.168.1.4",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.5",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.6",
        "unavailable-tcp-ports": [
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88
        ],
        "available-tcp-ports": [
          80,
          443
        ],
        "is-host-reachable": true,
        "unavailable-udp-ports": [
          50,
          51,
          52,
          53,
          69
        ]
      },
      {
        "host": "192.168.1.7",
        "is-host-reachable": false
      },
      {
        "host": "192.168.1.8",
        "is-host-reachable": false
      }
    ]
  }
}
```

### Successful example

RPC input contains the host name and ports to be checked for availability.

RPC output shows if the host is reachable and all open TCP/UDP ports.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "address": [
            {
                "hostname": "www.frinx.io"
            }
        ],
        "tcp-port": [
            {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "device": [
      {
        "host": "46.229.232.182",
        "available-udp-ports": [
          50,
          51,
          52,
          53,
          69
        ],
        "unavailable-tcp-ports": [
          81,
          82,
          83,
          84,
          85,
          86,
          87,
          88
        ],
        "available-tcp-ports": [
          80,
          443
        ],
        "is-host-reachable": true
      }
    ]
  }
}
```

### Failed example

RPC input contains two addresses that are incorrectly wrapped.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "address": [
            {
                "hostname": "www.frinx.io",
                "ip-address": "46.229.232.182"
            }
        ],
        "tcp-port": [
            {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "protocol",
        "error-tag": "malformed-message",
        "error-message": "Error parsing input: Data from case (urn:ietf:params:xml:ns:yang:rpc?revision=2021-07-08)ip-address-case are specified but other data from case (urn:ietf:params:xml:ns:yang:rpc?revision=2021-07-08)hostname-case were specified earlier. Data aren't from the same case.",
        "error-info": "Data from case (urn:ietf:params:xml:ns:yang:rpc?revision=2021-07-08)ip-address-case are specified but other data from case (urn:ietf:params:xml:ns:yang:rpc?revision=2021-07-08)hostname-case were specified earlier. Data aren't from the same case."
      }
    ]
  }
}
```

### Failed example

RPC input contains an IP range where the starting point is greater than the end
point.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "address": [
            {
                "start-ipv4-address": "127.0.0.63",
                "end-ipv4-address": "127.0.0.60"
            }
        ],
        "tcp-port": [
            {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "protocol",
        "error-tag": "bad-element",
        "error-message": "Invalid IP address range! End address should be bigger than start address! The range is from 127.0.0.63 to 127.0.0.60"
      }
    ]
  }
}
```

### Unsupported operation - example

RPC input contains a network in IPv6 format that is currently not supported.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/device-discovery:discover' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "address": [
            {
                "network": "2001:db8:a::123/64"
            }
        ],
        "tcp-port": [
            {
                "start-port": 80,
                "end-port": 88
            },
            {
                "port": 443
            }
        ],
        "udp-port": [
            {
                "port": 69
            },
            {
                "start-port": 50,
                "end-port":  53
            }
        ]
    }
}'
```

```json RPC Response, Status: 501
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-not-supported",
        "error-message": "IPv6 is not supported yet!"
      }
    ]
  }
}
```
