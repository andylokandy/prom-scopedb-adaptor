prom-scopedb-adaptor stores timeseries data in ScopeDB.

Implements both Prometheus remote writer (to store metrics) and
Prometheus remote reader (use metrics from ScopeDB directly in Prometheus)

### Install

```
go install github.com/andylokandy/prom-scopedb-adaptor@latest
```

### Create destination table

Use `scopeql` to create this table:

```
CREATE TABLE metrics.samples
(
    `updated_at` timestamp,
    `metric_name` string,
    `labels` Variant,
    `value` float,
)
CLUSTER BY (metric_name, labels, updated_at)
```

### Configure Prometheus remote writer

In your `prometheus.yaml`:

```
remote_write:
 - url: "http://localhost:9131/write"
   queue_config:
     max_samples_per_send: 10000
```

It prefers fewer writes with more samples per write.  Above a certain
rate you may need to adjust Prometheus `capacity` and
`max_samples_per_send` as per [Prometheus Remote Write Tuning](https://prometheus.io/docs/practices/remote_write/) if you see "Too many parts" errors or `prometheus_remote_storage_samples_pending` keeps growing.

### Configure Prometheus remote reader

In your `prometheus.yaml`:

```
remote_read:
 - url: "http://localhost:9131/read"
```

### Query data with Prometheus

The above configuration will use `prom-scopedb-adaptor` to backfill
data not present in Prometheus.

If you'd like to query `prom-scopedb-adaptor` immediately, consider
this configuration:

```
remote_read:
 - url: "http://localhost:9131/read"
   read_recent: true
   name: scopedb
   required_matchers:
     remote: scopedb
```

Then issue queries with the added `{remote="scopedb"}` label.

`prom-scopedb-adaptor` will remove the `{remote="scopedb"}` label
from incoming requests by default, see `--help`.
