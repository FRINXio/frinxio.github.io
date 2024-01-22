# Performance characteristics

This page contains reference performance characteristics for Uniconfig.

We try to answer the question how fast can a certain number of devices with a certain amount of configuration
be installed and fully synced by
- a single Uniconfig instance
- 3-node Uniconfig deployment with load balancer

The unit of measurement is: Number of configuration lines / per single CPU core / per minute.
This number can then be roughly applied to any other similar device being installed by uniconfig.

## CLI devices

There are 2 main families of CLI devices: those using Cisco style configuration (configuration in sections)
and devices that use one-line style of configuration (without sections) such as Ciena SAOS 8.

It is important to distinguish performance characteristics of these 2 families. 

## Tree-like style of configuration
Cisco style of confituration (IOS, IOS-XR etc.)

// TBD

## One-line style of configuration devices (SAOS) performance tests

Important caveats:
- Measurement were performed on simulated devices = no device overhead
- Measurement were performed on a local network = no network overhead
- Measured on Uniconfig version 5.0.12
- Simulated devices were of two flawors: half with small configuration and the half with big configuration

Tests:
- A. Single node deployment of Uniconfig resources: CPU 4 cores and RAM 4 GB
- B. 3 node deployment of Uniconfig - resources per node: CPU 4 cores and RAM 4 GB
- C. 3 node deployment of Uniconfig - resources per node: CPU 6 cores and RAM 4 GB

### Device installation & synchronization 

                        "input": { 
                            "cli": {
                                "cli-topology:device-type": "saos",
                                "cli-topology:device-version": "6",
                                "cli-topology:dry-run-journal-size": 180,
                                "cli-topology:host": "172.16.62.119",
                                "cli-topology:journal-size": 500,
                                "cli-topology:keepalive-delay": "900",
                                "cli-topology:keepalive-timeout": "3600",
                                "cli-topology:parsing-engine": "one-line-parser",
                                "cli-topology:password": "*****",
                                "cli-topology:port": "22",
                                "cli-topology:transport-type": "ssh",
                                "cli-topology:username": "*****",
                                "node-extension:reconcile": false,
                                "uniconfig-config:install-uniconfig-node-enabled": true
                            },
                            "node-id": "ASN-RC0002-DS119"
                        }

                        "input": { 
                            "cli": {
                                "cli-topology:device-type": "saos",
                                "cli-topology:device-version": "8",
                                "cli-topology:dry-run-journal-size": 180,
                                "cli-topology:host": "172.16.84.86",
                                "cli-topology:journal-size": 500,
                                "cli-topology:keepalive-delay": "900",
                                "cli-topology:keepalive-timeout": "3600",
                                "cli-topology:parsing-engine": "one-line-parser",
                                "cli-topology:password": "*****",
                                "cli-topology:port": "22",
                                "cli-topology:transport-type": "ssh",
                                "cli-topology:username": "*****",
                                "node-extension:reconcile": false,
                                "uniconfig-config:install-uniconfig-node-enabled": true
                            },
                            "node-id": "ASN-RC0002-CS598"
                        }
 

### Test A - one node Uniconfig
Devices running SAOS operating system (Ciena) and similar

Inputs:  
375 x SAOS 6 devices configuration:
8834 json lines = 1510 cli config lines (brief config)

375 x SAOS 8 devices configuration:
277375 json lines = 30705  cli config lines (brief config)

Evaluation:
750 devices were registered in 7.5 hours on single node Uniconfig using 4 cores  
Average one device instalation duration = (7.5 * 60 minutes) / 750 devices = 0.6 minutes  
Average number of json lines per device = (8834 + 277375)/2 = 143104 lines  
lines of json / per core / per minute = 143104 lines / 4 cores / 0.6 minutes = 59626  
Average number of cli lines per device = (1510 + 30705)/2 = 16107,5 lines  
lines of cli / per core / per minute = 16107,5 lines / 4 cores / 0.6 minutes = 6711  

Installation & sync rate:

**59,626** *lines of json / per core / per minute*

or

**6,711** *lines of raw cli configuration / per core / per minute*

> A single uniconfig node is capable of installing (and fully syncing)
> **100 Ciena (SAOS 8) devices with 15k lines of configuration** (~ 123k lines of formatted json in Uniconfig)
> in **55 minutes** using **4 CPU cores**

Recommended batch size for parallel installation in such case would be about 50 devices per batch as the parallelism is limited by the number of available cores.

### Test B - 3 nodes of Uniconfig with load balancer

Devices running SAOS operating system (Ciena) and similar

Inputs:  
750 x SAOS 6 devices configuration:
8834 json lines = 1510 cli config lines (brief config)

750 x SAOS 8 devices configuration:
277375 json lines = 30705  cli config lines (brief config)

Evaluation for 4 core deployment:
1500 devices were registered in 5.5 hours on 3 node Uniconfig deployment each using 4 cores  
Average one device instalation duration = (5.5 * 60 minutes) / 1500 devices = 0.22 minutes  
Average number of json lines per device = (8834 + 277375)/2 = 143104 lines  
lines of json / per core / per minute = 143104 lines / 3*4 cores / 0.22 minutes = 54206  
Average number of cli lines per device = (1510 + 30705)/2 = 16107,5 lines  
lines of cli / per core / per minute = 16107,5 lines / 3*4 cores / 0.22 minutes = 6101  

Installation & sync rate:

**54,206** *lines of json / per core / per minute*

or

**6,101** *lines of raw cli configuration / per core / per minute*

> A 3 nodes of uniconfig with loadbalancer are capable of installing (and fully syncing)
> **100 Ciena (SAOS 8) devices with 15k lines of configuration** (~ 123k lines of formatted json in Uniconfig)
> in **19 minutes** using **12 CPU cores**

Recommended batch size for parallel installation in such case would be about 150 devices per batch as the parallelism is limited by the number of available cores.

### Test C - 3 nodes of Uniconfig with load balancer

Devices running SAOS operating system (Ciena) and similar

Inputs:  
750 x SAOS 6 devices configuration:
8834 json lines = 1510 cli config lines (brief config)

750 x SAOS 8 devices configuration:
277375 json lines = 30705  cli config lines (brief config)

Evaluation for 6 core deployment:
1500 devices were registered in 3.7 hours on 3 node Uniconfig deployment each using 6 cores  
Average one device instalation duration = (3.7 * 60 minutes) / 1500 devices = 0.148 minutes  
Average number of json lines per device = (8834 + 277375)/2 = 143104 lines  
lines of json / per core / per minute = 143104 lines / 3*6 cores / 0.148 minutes = 53717  
Average number of cli lines per device = (1510 + 30705)/2 = 16107,5 lines  
lines of cli / per core / per minute = 16107,5 lines / 3*6 cores / 0.148 minutes = 6046  

Installation & sync rate:

**53,717** *lines of json / per core / per minute*

or

**6,046** *lines of raw cli configuration / per core / per minute*

> A 3 nodes of uniconfig with loadbalancer are capable of installing (and fully syncing)
> **100 Ciena (SAOS 8) devices with 15k lines of configuration** (~ 123k lines of formatted json in Uniconfig)
> in **13 minutes** using **18 CPU cores**


Recommended batch size for parallel installation in such case would be about 150 devices per batch as the parallelism is limited by the number of available cores.

## Netconf devices

We use netconf-testtool to simulate devices.

Important caveats:
- Measurement were performed on simulated devices = no device overhead
- Measured on Uniconfig version 5.2.1
- Simulated devices were with small configuration (~ 200 json lines)

Tests:
- A. Single node deployment of Uniconfig resources: CPU 12 cores and RAM 4 GB

### Device installation & synchronization 
```
{  
    "input": {  
        "nodes": [  
            {  
                "node-id": "node-20820",  
                "netconf": {  
                    "netconf-node-topology:host": "10.19.0.253",  
                    "netconf-node-topology:port": "20820",  
                    "netconf-node-topology:keepalive-delay": 5,  
                    "netconf-node-topology:tcp-only": "False",  
                    "netconf-node-topology:username": "admin",  
                    "netconf-node-topology:password": "admin",  
                    "netconf-node-topology:dry-run-journal-size": 100,  
                    "netconf-node-topology:enabled-notifications": "False",  
                    "uniconfig-config:install-uniconfig-node-enabled": "True",  
                    "uniconfig-config:uniconfig-native-enabled": "True",  
                    "netconf-node-topology:max-connection-attempts": 1,  
                    "netconf-node-topology:connection-timeout-millis": 60000,  
                    "netconf-node-topology:default-request-timeout-millis": 60000,  
                    "netconf-node-topology:sleep-factor": "1.0"  
                }  
            }  
        ]  
    }  
}  
```

### Test A - one node Uniconfig

Inputs:  
5000 x Netconf devices simulated:
203 json lines

Evaluation 1:
5000 devices were registered in 16.5 minutes on single node Uniconfig using 12 cores  
Average one device instalation duration = 16.5 minutes / 5000 devices = 0,0033 minutes  
Average number of json lines per device = 203 lines  
lines of json / per core / per minute = 203 lines / 12 cores / 0,0033 minutes = 5126

Installation & sync rate:

**5,126** *lines of json / per core / per minute*

> A single uniconfig node is capable of installing (and fully syncing)
> **5000 netconf devices with config of 0.2k lines each of formatted json (in Uniconfig)**
> in **16.5 minutes** using **12 CPU cores**

5 threads were used and *connection-manager:install-multiple-nodes* RPC was used (with 20 devices each).


Evaluation 2:
5000 devices were updated in 12.4 minutes on single node Uniconfig using 12 cores  
Update process consisted of RPCs: Create transaction, Apply template, Calculate diff, Commit
After apply template each configuration was increased from 203 to 1282 json lines
Average one device update duration = 12.4 minutes / 5000 devices = 0,00248 minutes  
Average number of json lines per device = 1282 lines  
lines of json / per core / per minute = 1282 lines / 12 cores / 0,00248 minutes = 43077

Update rate:

**43,077** *lines of json / per core / per minute*

> A single uniconfig node is capable of updating (and fully syncing)
> **5000 netconf devices with initial config of 0.2k json lines each (of formatted json in Uniconfig)**
> in **12.4 minutes** using **12 CPU cores**
> After update each device has config of 1.2k json lines each

20 threads were used. No delay among successive updating RPCs


## GNMI devices

We use gnmi-testtool to simulate devices.

Important caveats:
- Measurement were performed on simulated devices = no device overhead
- Measured on Uniconfig version 5.2.5
- Simulated devices were with small configuration (~ 1144 json lines)
- Devices were installed using optimized RPC connection-manager:install-multiple-nodes (up to 250 devices)

Tests:
- A. Single node deployment of Uniconfig resources: CPU 12 cores and RAM 4 GB

### Device installation & synchronization 
```
{  
    "input": {  
        "nodes": [  
            {
              'node-id': 'node-36100',
              'gnmi': {
                'schema-cache-directory': 'gnmi-topology-testtool',
                'update-paths': [
                  '.+'
                ],
                'uniconfig-config:whitelist': {
                  'path': [
                    'openconfig-interfaces:interfaces',
                    'openconfig-system:system',
                    'openconfig-openflow:openflow'
                  ]
                },
                'uniconfig-config:uniconfig-native-enabled': True,
                'uniconfig-config:install-uniconfig-node-enabled': True,
                'uniconfig-config:sequence-read-active': True,
                'connection-parameters': {
                  'host': '10.19.0.253',
                  'port': '36100',
                  'connection-type': 'PLAINTEXT',
                  'credentials': {
                    'username': 'admin',
                    'password': 'password'
                  }
                },
                'session-timers': {
                  'request-timeout': 100,
                  'internal-transaction-timeout': 100
                },
                'extensions-parameters': {
                  'gnmi-parameters': {
                    'use-model-name-prefix': False
                  },
                  'force-cached-capabilities': [
                    'null'
                  ]
                },
                'stream': [
                  {
                    'stream-name': 'GNMI_testtool',
                    'paths': [
                      'openconfig-system:system/config',
                      'openconfig-system:system/aaa',
                      'openconfig-interfaces:interfaces/interface',
                      'openconfig-lldp:lldp'
                    ]
                  }
                ]
              }
            }
        ]  
    }  
}  
```

### Test A - one node Uniconfig

Inputs:  
6000 x GNMI devices simulated:
1144 json lines

Evaluation 1:
6000 devices were registered in 5.2 minutes on single node Uniconfig using 12 cores  
Average one device instalation duration = 5.2 minutes / 6000 devices ~ 0.00087 minutes  
Average number of json lines per device = 1144 lines  
lines of json / per core / per minute = 1144 lines / 12 cores / 0.00087 minutes = 109578

Installation & sync rate:

**109,578** *lines of json / per core / per minute*

> A single uniconfig node is capable of installing (and fully syncing)
> **6000 GNMI devices with config of 1.1k lines each of formatted json (in Uniconfig)**
> in **5.2 minutes** using **12 CPU cores**

RPC were sent serially - *connection-manager:install-multiple-nodes* RPC was used (with 100 devices each).


Evaluation 2:
Devices were PATCHED - after PATCH they contain 10 more interfaces - which is 1234 json lines.  
Then 6000 devices were updated in 4.3 minutes on single node Uniconfig using 12 cores.  
Update process consisted of RPCs: sync-from-network
After sync-from-network each configuration was increased from 1144 to 1234 json lines
Average one device update duration = 4.3 minutes / 6000 devices = 0,00072 minutes  
Average number of json lines per device = 1234 lines  
lines of json / per core / per minute = 1234 lines / 12 cores / 0,00072 minutes = 142824

Update rate:

**142,824** *lines of json / per core / per minute*

> A single uniconfig node is capable of updating (and fully syncing)
> **6000 GNMI devices with config of 1.1k json lines each (of formatted json in Uniconfig)**
> in **4.3 minutes** using **12 CPU cores**
> After update each device has config of 1.2k json lines each

10 threads were used. No delay among successive updating RPCs. RPC sync-from-network with 60 devices each RPC. 
