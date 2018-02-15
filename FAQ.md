### Most common reasons why Prometheus is OOM killed.
The obvious one - not enough memory to handle the ingestion or querying rate.
Ingesting too many short lived time series with a very high cardinality.

### Can I use Prometheus for short lived time series with a very high cardinality?
This use case fits better with a logging system like ELK. Prometheus is not an event logging system, and attempting to use it that way will result in low performance and other issues.

### How do I do backups?
There is a very good [article by Robust perception](https://www.robustperception.io/taking-snapshots-of-prometheus-data/) which should answer this question.

### Why Prometheus uses so much memory?
Prometheus's new storage engine memory-maps the entire storage. On a machine with lots of free memory this will make it appear like memory usage keeps growing indefinitely. However, the memory will be released as soon as another process needs it.

### How to handle configuration management?
Prometheus intends to be a monitoring system, not to duplicate the functionality of configuration management.<br/>
For Kubernetes environment [ksonnet](https://ksonnet.io/) would be a popular choice.

