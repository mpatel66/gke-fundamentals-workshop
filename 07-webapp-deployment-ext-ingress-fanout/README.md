# GKE Workshop LAB-07

## Web-Application Deployment, GCE-FanOut-Ingress Example

[![Context](https://img.shields.io/badge/GKE%20Fundamentals-1-blue.svg)](#)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![GKE/K8s Version](https://img.shields.io/badge/k8s%20version-1.18.20-blue.svg)](#)
[![GCloud SDK Version](https://img.shields.io/badge/gcloud%20version-359.0.0-blue.svg)](#)

## Introduction

In the following lab we will set up our local development environment, provision the workshop cluster and roll out three different versions of our static web application example ([source-v1](https://github.com/doitintl/labs-web-app-static), [source-v2](https://github.com/doitintl/labs-web-app-static-blue), [source-v3](https://github.com/doitintl/labs-web-app-static-green)). Those deployments will be exposed by corresponding pod-services and wait for incoming traffic through our GCE Ingress-Controller. The different versions will be available by path-based routing through an external load-balancer (`http://<external-lb-ip>/`,`http://<external-lb-ip>/v2/` and `http://<external-lb-ip>/v3/`).

|                              default-version                               |                             blue-version (/v2)                             |                            green-version (/v3)                             |
| :------------------------------------------------------------------------: | :------------------------------------------------------------------------: | :------------------------------------------------------------------------: |
| ![application screenshot](../.github/media/lab-07-screenshot-small-v1.png) | ![application screenshot](../.github/media/lab-07-screenshot-small-v2.png) | ![application screenshot](../.github/media/lab-07-screenshot-small-v3.png) |

## Cluster Application Deployment

1. Run deployment

```bash
kubectl apply -f .
```

2. Check current ingress state (external IP)

_this step can take up to 3 Minutes_

```bash
kubectl get ingress -n doit-lab-07 --watch
```

## Cluster Application Check / Playground

1. You can check the state of Pods at any time with the following kubectl command:

```bash
kubectl get pods -n doit-lab-07
```

2. You can check your ingress target service with the following kubectl command:

```bash
kubectl get service -n doit-lab-07
```

3. You can get some more detailed information about your ingress resource by the following kubectl command:

```bash
kubectl describe ingress static-web-app-ingress -n doit-lab-07
```

| Path | Backends                                                                   |
| ---- | -------------------------------------------------------------------------- |
| /    | static-web-app-service-v1:8080 (10.72.0.13:80,10.72.1.39:80,10.72.2.17:80) |
| /v2/ | static-web-app-service-v2:8080 (10.72.0.14:80,10.72.1.40:80,10.72.2.16:80) |
| /v3/ | static-web-app-service-v3:8080 (10.72.0.15:80,10.72.1.41:80,10.72.2.18:80) |

4. As soon as your ingress resource, ingress controller and the corresponding loadBalancer is provisioned (3-4 minutes):

4.1 You can check the benchmark of your web-application using apache-bench command as shown below:

```bash
ab -n 20 http://<external-ip-of-your-ingress-load-balancer>/
http://34.111.29.162/

```

4.2 You can simulate some traffic to your ingress facing loadBalancer by the following ab-command:

```bash
ab -n 500 -c 25 http://<external-ip-of-your-ingress-load-balancer>/
```

4.3 Or just visit the three different application versions by hitting their ingress-path definition:

```bash
  -> version_1 - http://<external-ip-of-your-ingress-load-balancer>/
  -> version_2 - http://<external-ip-of-your-ingress-load-balancer>/v2/
  -> version_3 - http://<external-ip-of-your-ingress-load-balancer>/v3/
```

## Optional Steps

Now we can set the current k8s context to our lab exercise namespace `doit-lab-07` to make sure that every command set is run against this lab resources.

```bash
kubectl config set-context --current --namespace=doit-lab-07
```

In this example we can access the authentication token with a much shorter command line (we just ignore the namespace property now).

```bash
kubectl describe ingress static-web-app-ingress
```

## Application Clean-Up

```bash
kubectl delete -f .
```

## Links

- https://cloud.google.com/sdk/gcloud/reference/container/clusters/create
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://phoenixnap.com/kb/kubectl-commands-cheat-sheet

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

## Copyright

Copyright © 2021 DoiT International
