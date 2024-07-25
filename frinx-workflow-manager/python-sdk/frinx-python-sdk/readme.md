---
order: 4000
label: Frinx Python SDK
---

Frinx Python SDK
==========================

The FRINX Python SDK is a flexible tool designed to simplify interaction with the FRINX network automation solutions.
This SDK provides a set of Python libraries and utilities that enable developers to easily integrate with FRINX's platform, 
streamline their network automation workflows, and leverage FRINX's capabilities to the fullest.

## Project init

For the beggining, you will need to create new python project.
Example of simple project can be found [here](https://github.com/FRINXio/frinx-python-sdk/tree/main/examples/simple_worker)
We recommend to use poetry as a package management tool and use package from [pypi.org](https://pypi.org/project/frinx-python-sdk/)


### pyproject.toml

```bash
# pyproject.toml
[tool.poetry]
name = "example_worker"
version = "0.0.1"
description = "Example of worker implementation"

[tool.poetry.dependencies]
python = "^3.10"
frinx-python-sdk = "^2"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### Worker definition

In worker.py we want to create some piece of logic, which will be used in workflows to solve complex problems.
Our worker has predefined interface, where you can define conductor task definitions, inputs and outpus.
In method execute, you can make any your custom logic. For example call to our service, parse data from previous task or any other code you want.


```python
# workers.test_worker.py
# imports from frinx-python-sdk
from frinx.common.conductor_enums import TaskResultStatus
from frinx.common.type_aliases import ListAny
from frinx.common.worker.service import ServiceWorkersImpl  
from frinx.common.worker.task_def import TaskDefinition
from frinx.common.worker.task_def import TaskInput
from frinx.common.worker.task_def import TaskOutput
from frinx.common.worker.task_result import TaskResult
from frinx.common.worker.worker import WorkerImpl

# ServiceWorkersImpl: Group workers based on purpose to one class. Then all workers can be registered together
class TestWorkers(ServiceWorkersImpl):

    # WorkerImpl: Specific worker implementation with predefined interfaces
    class Echo(WorkerImpl):

        # TaskDefinition: Define conductor task parameters 
        # https://docs.conductor-oss.org/documentation/configuration/taskdef.html
        class WorkerDefinition(TaskDefinition):
            name: str = 'TEST_echo'
            description: str = 'testing purposes: returns input unchanged'
            labels: ListAny = ['TEST']
            timeout_seconds: int = 60
            response_timeout_seconds: int = 60

        # TaskInput: Define conductor task input parameters
        # These parameters are validated via pydantic before execute method
        class WorkerInput(TaskInput):
            input: str

        # TaskOutput: Define conductor task output values
        class WorkerOutput(TaskOutput):
            output: str

        # Execute method contain task logic. 
        # Method got input parameters and returning TaskResult object
        def execute(self, worker_input: WorkerInput) -> TaskResult[WorkerOutput]:
            print(worker_input.input)
            return TaskResult(
                status=TaskResultStatus.COMPLETED,
                logs=['Echo worker invoked successfully'],
                output=self.WorkerOutput(output=worker_input.input)
            )
```

### Workflow definition

Now let's use previous created worker in workflow. 

```python
# workers.test_workflow.py
# imports from frinx-python-sdk
from frinx.common.conductor_enums import TimeoutPolicy
from frinx.common.type_aliases import ListStr
from frinx.common.workflow.service import ServiceWorkflowsImpl
from frinx.common.workflow.task import SimpleTask
from frinx.common.workflow.task import SimpleTaskInputParameters
from frinx.common.workflow.workflow import FrontendWFInputFieldType
from frinx.common.workflow.workflow import WorkflowImpl
from frinx.common.workflow.workflow import WorkflowInputField


# ServiceWorkflowsImpl: Group workflows based on purpose to one class. Then all workflows can be registered together
class TestWorkflows(ServiceWorkflowsImpl):

    # WorkflowImpl: Specific workflow implementation with predefined interfaces
    # https://docs.conductor-oss.org/documentation/configuration/workflowdef/index.html
    class TestWorkflow(WorkflowImpl):
        name: str = 'Test_workflow'
        version: int = 1
        description: str = 'Test workflow built from test workers'
        labels: ListStr = ['TEST']
        timeout_seconds: int = 60 * 5
        timeout_policy: TimeoutPolicy = TimeoutPolicy.TIME_OUT_WORKFLOW

        # WorkflowImpl.WorkflowInput: Define workflow inputs
        # WorkflowInputField: Define, how input will be rendered in UI during execution 
        class WorkflowInput(WorkflowImpl.WorkflowInput):
            text: WorkflowInputField = WorkflowInputField(
                name='text',
                frontend_default_value='hello world',
                description='How many seconds to sleep during the workflow',
                type=FrontendWFInputFieldType.STRING,
            )

        # WorkflowImpl.WorkflowOutput: Define workflow outputs
        class WorkflowOutput(WorkflowImpl.WorkflowOutput):
            text: str

        # workflow_builder method helps you build workflow JSON representation via code
        # you can make any logic here to handle various situation
        # for more info, how to create workflows see workflow-builder docs
        # https://docs.conductor-oss.org/devguide/labs/first-workflow.html
        def workflow_builder(self, workflow_inputs: WorkflowInput) -> None:

            echo_task = SimpleTask(
                name=TestWorkers.Echo,
                task_reference_name='echo',
                input_parameters=SimpleTaskInputParameters(
                    root=dict(
                        input=workflow_inputs.text.wf_input()
                    )
                )
            )
            self.tasks.append(echo_task)

```

### Start worker

Now make conductor client, which register our workflow and will be executing custom worker logic. 

```python
# main.py
import logging
import os

# Import conductor client
from frinx.client.v2.frinx_conductor_wrapper import FrinxConductorWrapper
from frinx.common.logging import logging_common
from frinx.common.logging.logging_common import LoggerConfig
from frinx.common.logging.logging_common import Root


# Register your tasks
def register_tasks(conductor_client: FrinxConductorWrapper) -> None:
    logging.info('Register tasks')
    from workers.test_worker import TestWorkers
    TestWorkers().register(conductor_client=conductor_client)

# Register your workflows
def register_workflows() -> None:
    logging.info('Register workflows')
    from workers.test_workflow import TestWorkflows
    TestWorkflows().register(overwrite=True)


def main() -> None:

    # Enable logging
    logging_common.configure_logging(
        LoggerConfig(
            root=Root(
                level=os.environ.get('LOG_LEVEL', 'INFO').upper(),
                handlers=['console']
            )
        )
    )

    # Enable prometheus metrics
    from frinx.common.telemetry.metrics import Metrics
    from frinx.common.telemetry.metrics import MetricsSettings

    Metrics(settings=MetricsSettings(metrics_enabled=True))

    # Register conductor client
    from frinx.common.frinx_rest import CONDUCTOR_HEADERS
    from frinx.common.frinx_rest import CONDUCTOR_URL_BASE

    conductor_client = FrinxConductorWrapper(
        server_url=CONDUCTOR_URL_BASE,
        # Define polling interval for conductor client. 
        # How often client will ask conductor for task to execute
        polling_interval=float(os.environ.get('CONDUCTOR_POLL_INTERVAL', 0.1)),
        # Number of parallel threads, which can execute tasks
        max_thread_count=int(os.environ.get('CONDUCTOR_THREAD_COUNT', 50)),
        headers=dict(CONDUCTOR_HEADERS),
    )

    register_tasks(conductor_client)
    register_workflows()
    conductor_client.start_workers()


if __name__ == '__main__':
    main()

```

