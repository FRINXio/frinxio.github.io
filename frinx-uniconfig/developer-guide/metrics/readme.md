# Metrics

## Monitoring Uniconfig performance

[Micrometer Metrics](https://micrometer.io/) is the framework of choice to monitor performance.

## Registry naming

All the metrics are currently stored in the global registry.
It can be accessed like so:
```java
import io.micrometer.core.instrument.Metrics;

Metrics.globalRegistry;
```

## Metric types

All the available metric types can be seen in the [documentation](https://docs.micrometer.io/micrometer/reference/concepts.html).

## Naming convention

There are various best practice articles on how to name metrics but one thing is common: It should be clear what is measured.

```java
import io.micrometer.core.instrument.Metrics;

Metrics.globalRegistry.counter(RpcResult.class.getName() + ".rpc_invoke");
```

## Adding new metrics

### Adding a Counter

Obtain a Counter and then increment all the method calls you want to measure.

```java
private final Counter rpcCounter = Metrics.globalRegistry.counter(RpcResult.class.getName() + ".rpc_invoke");

private void foo() {
    rpcCounter.increment();
}
```

### Adding a Gauge

Here we create a **Gauge** that returns Integer value, access is synchronized in this case to avoid race conditions.

```java
Metrics.globalRegistry.gauge(UniconfigTransactionManager.class.getName() + ".open_transaction_count",
        this, uniconfigTransactionManager -> {
            synchronized (UniconfigTransactionManagerImpl.this) {
                return UniconfigTransactionManagerImpl.this.uniconfigTransactions.size();
            }
        });
```

## Reporters 

Metrics are available at /actuator/prometheus endpoint.