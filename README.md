# Cheapsk8s

Cheap Kuberernetes compute

## Overview

Cheapsk8s hunts through cloud providers to find the cheapest compute for your workload. 

### Find compute based on resource requests
```sh
$ cks find --cpu 2 --mem 50kb --storage 100gi

select option:

a. aws ec2 spot instance -- single node cluster (us-west-2) -- $0.10/hr
b. gce preemptible vm -- single node cluster (us-central1) -- $0.12/hr
c. azure spot instance -- single node cluster (west-us) -- $0.09/hr

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
          image: perl
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
        restartPolicy: Never
    backoffLimit: 4
```
```
$ cks experiment -f my-experiment1.yaml
```

### Manage compute
```sh
$ cks compute view
NAME      PROVIDER     COST
aws-1     aws          $0.10/hr
gce-5     gce          $0.12/hr
```
Switch kubeconfigs
```sh
$ cks switch

select compute:

a. aws-1
b. gce-5
```
