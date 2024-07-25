---
order: 3000
label: Frinx Servives Python Workers
---

Frinx Servives Python Workers
==========================

The FRINX Services Python Workers repository contains a collection of commonly used workers and API wrappers. 
These components are designed to facilitate rapid workflow development, enabling developers to share tasks across projects and avoid redundant work. 
Contributions are welcome to help expand and improve the repository.

For more details, visit the [FRINX Services Python Workers documentation](https://github.com/FRINXio/frinx-services-python-workers).

### Package Importing

To import the necessary FRINX services into your project, add the following entries to your pyproject.toml file:

``` bash 
#pyproject.toml
[tool.poetry]
name = "example-project"
version = "0.1.0"
description = ""
readme = "README.md"
packages = []

[tool.poetry.dependencies]
python = "^3.10"
frinx-inventory-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "inventory/python"}
frinx-schellar-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "schellar/python"}
frinx-uniconfig-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "uniconfig/python"}
frinx-resource-manager-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "resource-manager/python"}
frinx-topology-discovery-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "topology-discovery/python"}
frinx-http-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "misc/python/http"}
frinx-kafka-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "misc/python/kafka"}
frinx-influxdb-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "misc/python/influxdb"}
frinx-conductor-system-test-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "misc/python/conductor-system-test"}
frinx-python-lambda = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "6.1.0", subdirectory = "misc/python/python-lambda"}

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

Each package can be imported via frinx_worker. Below is an example of how to use the UniconfigManager from the frinx-uniconfig-worker package:

### Example Usage

UniconfigManager worker implementation can be found in the [UniconfigManager](https://github.com/FRINXio/frinx-services-python-workers/blob/6.1.0/uniconfig/python/frinx_worker/uniconfig/uniconfig_manager.py) module.

In this example, the frinx-uniconfig-worker dependency is imported into the project, and the UniconfigManager module is used to manage Uniconfig transactions.
This approach allows you to use pre-built tasks instead of creating custom workers.

```python
from frinx_worker.uniconfig.uniconfig_manager import UniconfigManager

...

        def workflow_builder(self, workflow_inputs: WorkflowInput) -> None:
            tx_start = SimpleTask(
                name=UniconfigManager.CreateTransaction,
                task_reference_name="tx_start",
                input_parameters=SimpleTaskInputParameters(root=dict(uniconfig_url_base=workflow_inputs.zone.wf_input)),
            )

            commit_tx = SimpleTask(
                name=UniconfigManager.CommitTransaction,
                task_reference_name="tx_commit",
                input_parameters=SimpleTaskInputParameters(
                    root=dict(
                        uniconfig_url_base=tx_start.output_ref("uniconfig_url_base"),
                        uniconfig_server_id=tx_start.output_ref("uniconfig_server_id"),
                        transaction_id=tx_start.output_ref("transaction_id"),
                    )
                ),
            )
...

```


### Versioning

We release versions for each component change or as a combination of services based on the FRINX Machine release. 
If you have deployed FRINX Machine 6.1.0, use tag 6.1.0. For specific dependency versions, use the custom tag/branch/revision as shown below:


```bash
[tool.poetry.dependencies]
...
frinx-inventory-worker = {git = "ssh://git@github.com/FRINXio/frinx-services-python-workers.git", tag = "frinx-inventory-worker_v1.0.1", subdirectory = "inventory/python"}
...
```

By following these instructions, you can quickly integrate FRINX Services Python Workers into your project and start utilizing the provided functionalities to streamline your workflow development.

### 