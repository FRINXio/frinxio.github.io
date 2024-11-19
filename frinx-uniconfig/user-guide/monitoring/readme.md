## Monitoring using Metrics

UniConfig exposes multiple metrics for traffic and performance monitoring.

Output is generated in plaintext format:
- Plaintext response output from actuator/prometheus endpoint.

## Types of metrics
- Gauge - Reports the instantaneous value at a point in time (for example, queue size).
- Counter - Measures the total count for event occurrences.

## List of notable metrics exposed by UniConfig

- **Gauges**
    - `io.frinx.uniconfig.manager.impl.task.TaskExecutorImpl.queue_size` -
      Number of tasks in the queue waiting for execution
    - `org.apache.sshd.server.SshServer.active_sessions` -
      Number of active CLI sessions
    - `org.opendaylight.controller.uniconfig.transaction.manager.api.UniconfigTransactionManager.open_transaction_count` -
      Number of open transactions
- **Counters**
    - `org.opendaylight.yangtools.yang.common.RpcResult.rpc_invoke` -
      All RPCs invoked by UniConfig
    - `org.opendaylight.controller.uniconfig.transaction.manager.impl.UniconfigTransactionManagerImpl.transaction_invoke` -
      All transactions invoked in UniConfig
    - `io.frinx.uniconfig.shell.cli.SshTerminal.cli_message` -
      All commands invoked in UniConfig shell

## Configuration

Configuration is performed via a section in the **application.properties** file:

```properties
# flag that determines if metrics will be reported or not
management.endpoint.prometheus.enabled=true
```

## Example output

- **example prometheus endpoint output, http://127.0.0.1:8181/actuator/prometheus**:

```text
org_opendaylight_controller_uniconfig_transaction_manager_impl_UniconfigTransactionManagerImpl_transaction_invoke_total 125.0
org_opendaylight_controller_uniconfig_transaction_manager_api_UniconfigTransactionManager_open_transaction_count 57.0
org_opendaylight_yangtools_yang_common_RpcResult_rpc_invoke_total 103.0
io_frinx_uniconfig_manager_impl_task_exec_UniconfigTaskExecutor_queue_size 54.0
```