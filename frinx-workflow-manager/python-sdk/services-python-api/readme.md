---
order: 4000
label: Frinx Servives Python API
---

Frinx Servives Python API
==========================

The FRINX Services Python API repository is a monorepo containing Pydantic API wrappers. 
These components are designed to facilitate rapid worker development, enabling developers to use a common source of API and track changes between service releases. 
If you find any incompatibilities, please create an issue on the GitHub repository.

For more details, visit the [FRINX Services Python API documentation](https://github.com/FRINXio/frinx-services-python-api).

### Package Importing

To import the necessary FRINX API modules into your project, add the following entries to your pyproject.toml file:

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
pydantic = "^2"
requests = "^2.31.0"
frinx-python-sdk = "^2"
frinx-uniconfig-api = { git = "https://github.com/FRINXio/frinx-services-python-api.git", tag = "6.1.0", subdirectory = "uniconfig/python" }

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

Each package can be imported via frinx_api. Below is an example of how to create custom worker with imported API module:

### Example Usage

CreateTransaction API wrapper can be found in the [Uniconfig](https://github.com/FRINXio/frinx-services-python-api/blob/main/uniconfig/python/frinx_api/uniconfig/rest_api.py) module.

In this example, the frinx-uniconfig-api dependency is imported into the project, and the CreateTransaction module is used to manage Uniconfig transactions.
This approach allows you to use generated API instead of creating custom API implementation.

```python
import requests
from frinx.common.frinx_rest import UNICONFIG_HEADERS
from frinx.common.frinx_rest import UNICONFIG_URL_BASE
from frinx.common.worker.task_def import TaskDefinition
from frinx.common.worker.task_def import TaskExecutionProperties
from frinx.common.worker.task_def import TaskInput
from frinx.common.worker.task_def import TaskOutput
from frinx.common.worker.task_result import TaskResult
from frinx.common.worker.task_result import TaskResultStatus
from frinx.common.worker.worker import WorkerImpl


class CreateTransaction(WorkerImpl):
    from frinx_api.uniconfig.rest_api import CreateTransaction as UniconfigApi

    class ExecutionProperties(TaskExecutionProperties):
        exclude_empty_inputs: bool = True
        transform_string_to_json_valid: bool = True

    class WorkerDefinition(TaskDefinition):
        name: str = "UNICONFIG_Create_transaction_RPC"
        description: str = "Create Uniconfig transaction"

    class WorkerInput(TaskInput):
        transaction_timeout: int | None = None
        use_dedicated_session: bool = False
        uniconfig_url_base: str = UNICONFIG_URL_BASE

    class WorkerOutput(TaskOutput):
        transaction_id: str | None = None
        uniconfig_server_id: str | None = None
        uniconfig_url_base: str

    def execute(self, worker_input: WorkerInput) -> TaskResult[WorkerOutput]:
        if self.UniconfigApi.request is None:
            raise Exception(f"Failed to create request {self.UniconfigApi.request}")

        response = requests.request(
            url=worker_input.uniconfig_url_base + self.UniconfigApi.uri,
            method=self.UniconfigApi.method,
            data=class_to_json(self.UniconfigApi.request()),
            headers=dict(UNICONFIG_HEADERS),
        )

        if not response.ok:
            return TaskResult(
                status=TaskResultStatus.FAILED,
                logs=response.content.decode("utf8"),
                output=self.WorkerOutput(uniconfig_url_base=worker_input.uniconfig_url_base),
            )

        return TaskResult(
            status=TaskResultStatus.COMPLETED,
            output=self.WorkerOutput(
                transaction_id=response.cookies["UNICONFIGTXID"],
                uniconfig_server_id=response.cookies.get("uniconfig_server_id", None),
                uniconfig_url_base=worker_input.uniconfig_url_base,
            )
        )

```

In this example, we used frinx-python-sdk and frinx-services-python-api to create conductor worker compatible with Frinx Machine 6.1.0 release.
For more detail, how to inport and execute this worke, please visit SDK part. 

### Versioning

We release versions for each component change or as a combination of services based on the FRINX Machine release. 
If you have deployed FRINX Machine 6.1.0, use tag 6.1.0. For specific dependency versions, use the custom tag/branch/revision as shown below:


```bash
[tool.poetry.dependencies]
...
frinx-inventory-api = {git = "ssh://git@github.com/FRINXio/frinx-services-python-api.git", tag = "frinx-inventory-api_v2.2.0", subdirectory = "frinx-inventory-server/python"}
...
```

By following these instructions, you can quickly integrate FRINX Services Python API into your project and start utilizing the provided functionalities to streamline your worker development.

###