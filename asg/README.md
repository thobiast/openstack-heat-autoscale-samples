# CPU-Based Instance Auto Scaling

An example of Instance CPU Auto Scaling.

This sample stack does not create Load Balancer resources. However, if the
variable "`lb_pool`" is specified, instances will be added to or removed from
the Load Balancer pool defined by that variable as needed.

Update the `environment.yaml` file to match your environment settings.


**Creating the stack**

```shell
$ openstack stack create -t ./template_asg.yaml -e ./environment.yaml my_stack
...

$ openstack stack resource list my_stack
...

$ openstack stack output show my_stack --all -f value
...
```
