### What is ETCD?
A distributed reliable key-value store that is Simple, Secure and Fast

- key-value store
	Unlike with a relational database the information that belongs to an object is stored separately from all other objects of the same kind. This means that changes to one object does not effect any other. 
	
	Ex: Employees vs Students

 1. Relational Database

| Name       | Location   | Salary | Grade |
| ---------- | ---------- | ------ | ----- |
| John Doe   | New York   | 5000   |       |
| Will Smith | New York   | 5200   |       |
| Bob Thomas | New Jersey |        | A     |
| Jill Klein | New York   |        | B     |

2. Key-value store
   
| Key      | Value    |
| -------- | -------- |
| Name     | John Doe |
| Location | New York |
| Salary   | 5000     | 

| Key      | Value      |
| -------- | ---------- |
| Name     | Bob Thomas |
| Location | New Jersey |
| Greade   | A          |

### Operate ETCD
We can manually interact with the key-value store per `etcdctl`. Here are some examples:
`etcdctl` can interact with the ETCD Server using 2 API versions v2 and v3. By default its set to v2. Each version has a seperate set of commands.

etcdctl v2 supports the following commands:
```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

etcdctl v3 commands:
```bash
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```

You can set the API version using the environment variable
```bash
export ETCDCTL_API=3
```
### ETCD in Kubernetes
Every information shown when the `kubectl get` is run comes from the ETCD. Every change made to the cluster such as:
- Joining additional nodes
- Deploying pods or replica-sets
Are updated in the ETCD Server. Only once this update has taken place, will it be considered complete.
