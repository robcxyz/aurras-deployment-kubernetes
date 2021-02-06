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

### License  
Licensed under [Apache-2.0](./LICENSE)