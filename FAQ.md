### Error loading config couldn't load configuration ... unmarshal errors ...
YAML is space sensitive so the most common problem is additional or missing spaces or tabs vs spaces.

### Error logs for "out-of-order series"
see [4108](https://github.com/prometheus/prometheus/issues/4108). Usually caused when more than one Prometheus server writes to the same data folder. <br/>
Can happen in k8s or docker swarm environments where more than one replica can be scheduled to mount the same data folder.

### Error logs for "too many files open"
A quick [google search](https://www.google.co.uk/search?q=how+to+increase+ulimit) should give you the answer how to increse the ulimit for your OS. <br/>
For the current session on linux you can use `ulimit -n 10048`

### Most common reasons why Prometheus is OOM killed.

The typical case is when a host system does not have enough memory to support a Prometheus scrape target configuration of a given size, but other contributing factors may include:

 - not enough memory to mmaps some required files .
 - high ingestion or querying rate.
 - ingesting too many short lived time series with a very high cardinality.

### Can I use Prometheus for short lived time series with a very high cardinality?
This use case fits better with a logging system like ELK. Prometheus is not an event logging system, and attempting to use it that way will result in low performance and other issues.

### How do I do backups?
There is a very good [article by Robust perception](https://www.robustperception.io/taking-snapshots-of-prometheus-data/) which should answer this question.

### Why Prometheus uses so much memory?
Prometheus's new storage engine memory-maps the entire storage. On a machine with lots of free memory this will make it appear like memory usage keeps growing indefinitely. However, the memory will be released as soon as another process needs it.

### How to handle configuration management?
Prometheus intends to be a monitoring system, not to duplicate the functionality of configuration management.<br/>
For Kubernetes environment [ksonnet](https://ksonnet.io/) would be a popular choice.

