---
title: Physical Cluster Replication Monitoring
summary: Monitor and observe replication streams between a primary and standby cluster.
toc: true
docs_area: manage
---

{{site.data.alerts.callout_info}}
{% include feature-phases/preview.md %}
{{site.data.alerts.end}}

{% include_cached new-in.html version="v23.2" %} You can monitor a physical cluster replication stream using:

- [`SHOW VIRTUAL CLUSTER ... WITH REPLICATION STATUS`](#sql-shell) in the SQL shell.
- The [Physical Replication dashboard](#db-console) on the DB Console.
- [Prometheus and Alertmanager](#prometheus) to track and alert on replication metrics.

## SQL Shell

In the standby cluster's SQL shell, you can query `SHOW VIRTUAL CLUSTER ... WITH REPLICATION STATUS` for detail on status and timestamps for planning [cutover]({% link {{ page.version.version }}/cutover-replication.md %}):

{% include_cached copy-clipboard.html %}
~~~ sql
 SHOW VIRTUAL CLUSTER standbyapplication WITH REPLICATION STATUS;
~~~

Refer to [Responses](#responses) for a description of each field.

{% include_cached copy-clipboard.html %}
~~~
id |        name        |     data_state     | service_mode | source_tenant_name |                                                     source_cluster_uri                                               | replication_job_id |        replicated_time        |         retained_time         | cutover_time
---+--------------------+--------------------+--------------+--------------------+----------------------------------------------------------------------------------------------------------------------+--------------------+-------------------------------+-------------------------------+---------------
3  | standbyapplication | replicating        | none         | application        | postgresql://{user}:{password}@{hostname}:26257/?options=-ccluster%3Dsystem&sslmode=verify-full&sslrootcert=redacted | 899090689449132033 | 2023-09-11 22:29:35.085548+00 | 2023-09-11 16:51:43.612846+00 |     NULL
(1 row)
~~~

### Responses

{% include {{ page.version.version }}/physical-replication/show-virtual-cluster-responses.md %}

#### Data state

{% include {{ page.version.version }}/physical-replication/show-virtual-cluster-data-state.md %}

## DB Console

You can access the [DB Console]({% link {{ page.version.version }}/ui-overview.md %}) for your standby cluster at `https://{your IP or hostname}:8080/`. Select the **Metrics** page from the left-hand navigation bar, and then select **Physical Cluster Replication** from the **Dashboard** dropdown. The user that accesses the DB Console must have `admin` privileges to view this dashboard.

{% include {{ page.version.version }}/ui/ui-metrics-navigation.md %}

{{site.data.alerts.callout_info}}
The **Physical Cluster Replication** dashboard tracks metrics related to physical cluster replication jobs. This is distinct from the [**Replication** dashboard]({% link {{ page.version.version }}/ui-replication-dashboard.md %}), which tracks metrics related to how data is replicated across the cluster, e.g., range status, replicas per store, and replica quiescence.
{{site.data.alerts.end}}

The **Physical Cluster Replication** dashboard contains three graphs for monitoring:

### Replication lag

<img src="{{ 'images/v23.2/ui-replication-lag.png' | relative_url }}" alt="DB Console Replication Lag graph showing results over the past hour" style="border:1px solid #eee;max-width:100%" />

The **Replication Lag** graph shows you the amount of time the replication is behind the current time.

Hovering over the graph displays:

- The date and time.
- The amount of time in seconds the replication is behind the current date and time.

### Logical bytes

<img src="{{ 'images/v23.2/ui-logical-bytes.png' | relative_url }}" alt="DB Console Logical Bytes graph showing results over the past hour" style="border:1px solid #eee;max-width:100%" />

The **Logical Bytes** graph shows you the throughput of the replicated bytes.

Hovering over the graph displays:

- The date and time.
- The number of logical bytes replicated in MiB.

{{site.data.alerts.callout_info}}
When you [start a replication stream]({% link {{ page.version.version }}/set-up-physical-cluster-replication.md %}#step-4-start-replication), the **Logical Bytes** graph will record a spike of throughput as the initial scan completes. {% comment %}link to technical details here{% endcomment %}
{{site.data.alerts.end}}

### SST bytes

<img src="{{ 'images/v23.2/ui-sst-bytes.png' | relative_url }}" alt="DB Console SST bytes graph showing results over the past hour" style="border:1px solid #eee;max-width:100%" />

The **SST Bytes** graph shows you the rate at which all [SST]({% link {{ page.version.version }}/architecture/storage-layer.md %}#ssts) bytes are sent to the [KV layer]({% link {{ page.version.version }}/architecture/storage-layer.md %}) by physical cluster replication jobs.

Hovering over the graph displays:

- The date and time.
- The number of SST bytes replicated in MiB.

## Prometheus

You can use Prometheus and Alertmanager to track and alert on physical cluster replication metrics. Refer to the [Monitor CockroachDB with Prometheus]({% link {{ page.version.version }}/monitor-cockroachdb-with-prometheus.md %}) tutorial for steps to set up Prometheus.

We recommend tracking the following metrics:

- `physical_replication.logical_bytes`: The logical bytes (the sum of all keys and values) ingested by all physical cluster replication jobs.
- `physical_replication.sst_bytes`: The [SST]({% link {{ page.version.version }}/architecture/storage-layer.md %}#ssts) bytes (compressed) sent to the KV layer by all physical cluster replication jobs.
- `physical_replication.replicated_time_seconds`: The [replicated time]({% link {{ page.version.version }}/physical-cluster-replication-technical-overview.md %}#cutover-and-promotion-process) of the physical replication stream in seconds since the Unix epoch.

## See also

- [DB Console Overview]({% link {{ page.version.version }}/ui-overview.md %})
- [Monitoring and Alerting]({% link {{ page.version.version }}/monitoring-and-alerting.md %})
