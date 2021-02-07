<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->
# Aurras Kubernetes Deployment Configuration
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)

### Prerequisites

1. [Kubernetes](https://kubernetes.io/)
2. [Helm](https://helm.sh/)

### Deployment Guide

Clone [aurras-deployment-kubernetes](https://github.com/HugoByte/aurras-deployment-kubernetes) with submodules

```text
git clone https://github.com/HugoByte/aurras-deployment-kubernetes --recurse-submodules
```

#### Openwhisk

1. Navigate to openwhisk setup directory

```text
cd aurras-deployment-kubernetes/openwhisk
```

2. Label worker nodes

* Windows or macOS

```text
kubectl label nodes --all openwhisk-role=invoker
```

* Ubuntu  
```text
kubectl label node kind-worker openwhisk-role=core
kubectl label node kind-worker2 openwhisk-role=invoker
```

3. Get InternalIP of the cluster  

* Windows or macOS  
```text
kubectl describe nodes | grep InternalIP
```

* Ubuntu  
```text
kubectl describe node kind-worker | grep InternalIP: | awk '{print $2}'
```

4. Creating mycluster.yaml with **apiHostName** as InternalIP of the nodes from the previous step.

```text
whisk:
  ingress:
    type: NodePort
    apiHostName: <INTERNAL-IP>
    apiHostPort: 31001

nginx:
  httpsNodePort: 31001
```

5. Create a namespace

```text
kubectl create namespace aurras
```

6. Deploy Openwhisk using helm

```text
helm install openwhisk ./helm/openwhisk -n aurras -f mycluster.yaml
```

7. Get the summary of installation using

```text
helm status openwhisk -n aurras
```
#### Event Feed - Substrate

1. Navigate to aurras-event-feed-substrate setup directory

```text
cd aurras-deployment-kubernetes/openwhisk
```

2. Get InternalIP of the cluster

```text
kubectl describe nodes | grep InternalIP
```

3. Creating mycluster.yaml with environment variables provided below with host of the **CHAIN\_ENDPOINT** and **OPENWHISK\_API\_HOST** as InternalIP of the nodes 

```text
env:
  - CHAIN_NAME: Node Template
  - CHAIN_ENDPOINT: ws://<INTERNAL-IP>:9944
  - LOGGERS: console,info;file,error,/logs/event-feed.log
  - EXCLUDES: system
  - TYPES_FILE: /configs/types.json #name of the types file should be types.json
  - KAFKA_BROKERS: kafka.docker:9092
  - KAFKA_TOPIC: node-template-topic
  - OPENWHISK_API_KEY: 23bc46b1-71f6-4ed5-8c54-816aa4f8c502:123zO3xZCLrMN6v2BKK1dXYFpXlPkccOFqm12CdAsMgRU4VrNZ9lyGVCGuMDGIwP
  - OPENWHISK_API_HOST: https://<INTERNAL-IP>:31001
  - OPENWHISK_NAMESPACE: guest
  - EVENT_RECEIVER: event-receiver
```

4. Add custom type if any for the chain to `helm/config/types.json`

5. Deploy Openwhisk using helm

```text
helm install aurras-event-feed-substrate ./helm -n aurras -f mycluster.yaml
```

6. Get the summary of installation using

```text
helm status aurras-event-feed-substrate -n aurras
```
### License  
Licensed under [Apache-2.0](./LICENSE)