- Run each node on a separate machine. Since CockroachDB replicates across nodes, running more than one node per machine increases the risk of data loss if a machine fails. Likewise, if a machine has multiple disks or SSDs, run one node with multiple `--store` flags and not one node per disk. For more details about stores, see [Start a Node]({% link {{ page.version.version }}/cockroach-start.md %}#store).

- When starting each node, use the [`--locality`]({% link {{ page.version.version }}/cockroach-start.md %}#locality) flag to describe the node's location, for example, `--locality=region=west,zone=us-west-1`. The key-value pairs should be ordered from most to least inclusive, and the keys and order of key-value pairs must be the same on all nodes.

- When deploying in a single availability zone:

    - To be able to tolerate the failure of any 1 node, use at least 3 nodes with the [`default` 3-way replication factor]({% link {{ page.version.version }}/configure-replication-zones.md %}#view-the-default-replication-zone). In this case, if 1 node fails, each range retains 2 of its 3 replicas, a majority.

    - To be able to tolerate 2 simultaneous node failures, use at least 5 nodes and [increase the `default` replication factor for user data]({% link {{ page.version.version }}/configure-replication-zones.md %}#edit-the-default-replication-zone) to 5. The replication factor for [important internal data]({% link {{ page.version.version }}/configure-replication-zones.md %}#create-a-replication-zone-for-a-system-range) is 5 by default, so no adjustments are needed for internal data. In this case, if 2 nodes fail at the same time, each range retains 3 of its 5 replicas, a majority.

- When deploying across multiple availability zones:

    - To be able to tolerate the failure of 1 entire AZ in a region, use at least 3 AZs per region and set `--locality` on each node to spread data evenly across regions and AZs. In this case, if 1 AZ goes offline, the 2 remaining AZs retain a majority of replicas.

    - To ensure that ranges are split evenly across nodes, use the same number of nodes in each AZ. This is to avoid overloading any nodes with excessive resource consumption.

- When deploying across multiple regions:

    - To be able to tolerate the failure of 1 entire region, use at least 3 regions.