### Why Prometheus uses so much memory?
Prometheus's new storage engine memory-maps the entire storage. On a machine with lots of free memory this will make it appear like memory usage keeps growing indefinitely. However, the memory will be released as soon as another process needs it.

### How to handle configuration management
Prometheus intends to be a monitoring system, not to duplicate the functionality of configuration management.<br/>
For Kubernetes environment [ksonnet](https://ksonnet.io/) would be a popular choice.

