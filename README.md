# cheapsk8s

Cheap Kuberernetes compute

## Overview

Cheapsk8s hunts through cloud providers to find the cheapest compute for your workload. 

### Find compute based on resource requests
```
$ cks find --cpu 2 --mem 50kb --storage 100gi

select option:

  aws ec2 spot instance -- single node cluster (us-west-2) -- $0.10/hr
> gce preemptible vm -- single node cluster (us-central1) -- $0.12/hr
  azure spot instance -- single node cluster (west-us) -- $0.09/hr

```
You can also find based on the set requests and limits in your k8s deployment configurations.
```sh
$ cks find -f my-deployment.yaml
```
or
```sh
$ cks find -f my-folder/
```

### Find compute based on runtime
Run a job that tests execution time vs cost on a variety of instance types

```yaml
apiVersion: cheapsk8s.io/v1
kind: Experiment
metadata:
  name: my-experiment1
spec:
  providers:
    aws:
      regions: [us-west-2, us-west-1]
      instances: [m4-large, t2-small]
    gce:
      zones: [us-central1]
      intances: [e2, c2]
  job:
    template:
      spec:
        containers:
        - name: pi
          image: pytorch
          command: ["python", "classifier_test.py"]
        restartPolicy: Never
    backoffLimit: 4
```
Create the experiment
```
$ cks experiment -f my-experiment1.yaml

running job in 6 different environments...
```
Retrieve the outcomes
```
$ cks experiment status my-experiment1 -o yaml

status:
  provider:
    aws:
      "us-west-2/m4-large":
        cost: $2.40
        time: 400s
      "us-west-1/t2-small":
        cost: $1.40
        time: 700s
    gce:
      "us-central1/ec2":
        cost: $2.39
        time: 400s
```
### Manage compute
List current clusters
```sh
$ cks compute view
NAME      PROVIDER     COST
aws-1     aws          $0.10/hr
gce-5     gce          $0.12/hr
```
Switch kubeconfigs
```
$ cks switch

select compute:

> aws-1
  gce-5
```
