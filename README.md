# OpenStack Heat template for CPU Auto Scaling

This repo contains sample templates to implement CPU auto scaling in
OpenStack using Aodh, Heat, Ceilometer, and Gnocchi.

The examples are updated to use the `cpu` metric instead of the deprecated `cpu_util`.
The `cpu` metric is
[cumulative (increasing over time) and expressed in nanoseconds (ns).](https://docs.openstack.org/ceilometer/latest/admin/telemetry-measurements.html)
To calculate the CPU threshold for a given utilization percentage, use the following formula:

> **Threshold = Percentage_in_Decimal × Flavor_Number_of_vCPUs × 1,000,000,000 × Granularity**

Notes:
- `Flavor_Number_of_vCPUs` refers to the number of vCPUs in the flavor used by the instance(s).
- `1,000,000,000` nanoseconds = 1 second.
- `Granularity` refers to the Gnocchi granularity of the CPU data.


### Example with 1 vCPU and a granularity of 300 seconds:

| Utilization (%) | Decimal Value of Pct | Calculation | Threshold |
| ---------------:| :-----------------:  | ----------: | ----: |
| 100% | 1   | 1   * 1 * 1000000000 * 300 | 300000000000.0
| 90%  | 0.9 | 0.9 * 1 * 1000000000 * 300 | 270000000000.0
| 80%  | 0.8 | 0.8 * 1 * 1000000000 * 300 | 240000000000.0
| 70%  | 0.7 | 0.7 * 1 * 1000000000 * 300 | 210000000000.0
| 60%  | 0.6 | 0.6 * 1 * 1000000000 * 300 | 180000000000.0
| 50%  | 0.5 | 0.5 * 1 * 1000000000 * 300 | 150000000000.0
| 40%  | 0.4 | 0.4 * 1 * 1000000000 * 300 | 120000000000.0
| 30%  | 0.3 | 0.3 * 1 * 1000000000 * 300 | 90000000000.0
| 20%  | 0.2 | 0.2 * 1 * 1000000000 * 300 | 60000000000.0
| 10%  | 0.1 | 0.1 * 1 * 1000000000 * 300 | 30000000000.0

### Example with 2 vCPUs and a granularity of 300 seconds:
| Utilization (%) | Decimal Value of Pct | Calculation | Threshold |
| ---------------:| :-----------------:  | ----------: | ----: |
| 100% | 1   | 1   * 2 * 1000000000 * 300 | 600000000000.0
| 90%  | 0.9 | 0.9 * 2 * 1000000000 * 300 | 540000000000.0
| 80%  | 0.8 | 0.8 * 2 * 1000000000 * 300 | 480000000000.0
| 70%  | 0.7 | 0.7 * 2 * 1000000000 * 300 | 420000000000.0
| 60%  | 0.6 | 0.6 * 2 * 1000000000 * 300 | 360000000000.0
| 50%  | 0.5 | 0.5 * 2 * 1000000000 * 300 | 300000000000.0
| 40%  | 0.4 | 0.4 * 2 * 1000000000 * 300 | 240000000000.0
| 30%  | 0.3 | 0.3 * 2 * 1000000000 * 300 | 180000000000.0
| 20%  | 0.2 | 0.2 * 2 * 1000000000 * 300 | 120000000000.0
| 10%  | 0.1 | 0.1 * 2 * 1000000000 * 300 |  60000000000.0


**To view the CPU usage of instances:**

```shell
$ openstack metric aggregates --resource-type instance --sort-column timestamp \
 '(metric cpu rate:mean)' server_group=<STACK_ID>
```

**To view the CPU percentage usage of instances:**

```shell
$ openstack metric aggregates --resource-type instance --sort-column timestamp \
 '(/ (* (/ (/ (metric cpu rate:mean) 1000000000) <GRANULARITY>) 100) <NUMBER_OF_vCPUs>)' \
 server_group=<STACK_ID>
```

Notes:
- Replace `<STACK_ID>` with your actual stack ID.
- Replace `<GRANULARITY>` with your granularity period in seconds.
- Replace `<NUMBER_OF_vCPUs>` with the number of vCPUs your instance flavor has.

In the output "Name" column, the first UUID represents the instance UUID.

**To view the average CPU percentage usage aggregated across all instances:**

```shell
$ openstack metric aggregates --resource-type instance --sort-column timestamp \
'(aggregate mean (/ (* (/ (/ (metric cpu rate:mean) 1000000000) <GRANULARITY>) 100) <NUMBER_OF_vCPUs>)))' \
server_group=<STACK_ID>
```

Notes:
- Replace `<STACK_ID>` with your actual stack ID.
- Replace `<GRANULARITY>` with your granularity period in seconds.
- Replace `<NUMBER_OF_vCPUs>` with the number of vCPUs in your instance flavor.


> The most recent entry may not be accurate if metrics for all instances are not yet available in Gnocchi.


Additional links:
- <https://docs.redhat.com/en/documentation/red_hat_openstack_platform/17.1/html-single/auto-scaling_for_instances/index>
- <https://docs.openstack.org/heat/latest/>
- <https://docs.openstack.org/ceilometer/latest/admin/telemetry-measurements.html>
- <https://docs.openstack.org/releasenotes/ceilometer/rocky.html>
- <https://github.com/openstack/heat-templates>
