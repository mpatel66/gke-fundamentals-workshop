# GKE Workshop LAB-13

## Scaling a deployment using HPA

[![Context](https://img.shields.io/badge/GKE%20Fundamentals-1-blue.svg)](#)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![GKE/K8s Version](https://img.shields.io/badge/k8s%20version-1.18.20-blue.svg)](#)
[![GCloud SDK Version](https://img.shields.io/badge/gcloud%20version-359.0.0-blue.svg)](#)

## Deployment

HPA operates on CPU/Mem usage as well as other custom/external metrics defined.

There is also VPA, which is based around usage rather than defining limits for the container. VPA can automatically update memory/cpu if the pod crashes due to limitations. It can also recommend values.

There is also multi-dimensional autoscaling which does both HPA and VPA. It's in beta.

1. Create namespace and deployments:

```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-php-deployment.yaml
```

2. Inspect the pods once they are running:

```bash
kubectl get pods -n doit-lab-13
kubectl describe pod ... -n doit-lab-13
```

3. Deploy the load generator:

```bash
kubectl apply -f 02-load-generator-pod.yaml
```

4. Check/wait until CPU and memory metrics are coming in:

```bash
watch -n 1 'kubectl top pod -n doit-lab-13'
```

The cpu of the pod is approaching 1000m, which is the limit defined in the HPA.

```bash
NAME                          CPU(cores)   MEMORY(bytes)
load-generator                14m          1Mi
php-apache-6b9ff8587f-lj9nl   862m         12Mi
```

5. Deploy the Horizontal Pod Autoscaler:

```bash
kubectl apply -f 03-php-hpa.yaml
```

Apply the HPA, and we see that 3 more pods are spawned as the limit of the first pod exceeds 1000m cpu.

6. Check/wait for scaleup:

```bash
watch -n 1 'kubectl top pod -n doit-lab-13'
```

```bash
NAME                          CPU(cores)   MEMORY(bytes)
load-generator                14m          1Mi
php-apache-6b9ff8587f-cdrfx   228m         12Mi
php-apache-6b9ff8587f-lj9nl   223m         12Mi
php-apache-6b9ff8587f-lwn22   193m         12Mi
php-apache-6b9ff8587f-m7cmx   173m         12Mi
```

Now we see tha the cpu usage for each of the pods is around 200m.

If we remove the load-generator-pod (which is churning out the requests to the pods), since the scale down takes 5 minutes to take effect, the extra pods do not tear down until after that period is up. However, we can see that the cpu usage is low.

```bash
NAME                          CPU(cores)   MEMORY(bytes)
php-apache-6b9ff8587f-cdrfx   1m           12Mi
php-apache-6b9ff8587f-lj9nl   1m           12Mi
php-apache-6b9ff8587f-lwn22   1m           12Mi
php-apache-6b9ff8587f-m7cmx   1m           12Mi
```

Since the scale down says reduce the number of pods by 25%, only one pod terminates after 5 minutes. The others terminate sequentially every 5 minutes until we're down to one pod.

## Application Clean-Up

```bash
kubectl delete -f .
```
