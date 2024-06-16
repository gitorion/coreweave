+++
title = '🦮 A guide to setup Stable Diffusion using Kubernetes on CoreWeave'
date = 2024-06-14T14:22:31+01:00
draft = false
summary = "Step-by-step instructions on setting up Stable Diffusion using a CoreWeave Namespace."
+++


## ✍️ CoreWeave Stable Diffusion Deployment Guide

### What is Kubernetes?

If this is your first time using Kubernetes, we can summarise Kubernetes as a container orchestrator. 
We have containers that provide us with the ability of deploying microservices, instead of overloading servers with multiple services, or wasting resources by deploying a single service on a clunky virtual machine (VM).

Instead, we can use containers to deploy a single service into a container. However, what if we had 200 services and needed to somehow manage those services? We could use Docker but the overhead would be very cumbersome to manage.
Not to mention that it would only work on a single hosts, what if we had 50 hosts? 

In comes Kubernetes, where we can create a Kubernetes environment that has hosts and we can deploy containers (pods) onto those hosts. This will give us reliability, potential for load balancing, increased security and even failover possibilities.

For more information regarding Kubernetes, please visit: [kubernetes.io: Overview](https://kubernetes.io/docs/concepts/overview/)

### Local Setup

In this section, we will look at setting up our local machine for being able to talk to CoreWeave's Kubernetes infrastructure. This guide already assumed that we have a running Namespace.

We will need to install `kubectl` first as this is the primary tool that we use to speak to Kubernetes. Issue the following command to see if it is available:

```shell
▶ kubectl version
```

##### Example output:

```shell
▶ kubectl version
Client Version: v1.30.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.20.15
```

---

If no current version is found install the kubectl package with the command relevant to your system:

- For Linux distributions see:
    https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

- For Windows see:
    https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

---

For the purposes of this guide and deployment, we will be using and assuming MacOS, therefore we can issue the following command to acquire `kubectl`:

> ℹ️ This assumes you have Homebrew installed on the local system, if not, we can acquire it here: [Homebrew: The Missing Package Manager for macOS](https://brew.sh)

```shell
▶ brew install kubectl

```

Once the utility has been installed, we will need to go and ensure that our CoreWeave kubeconfig has been put into the correct location so that we do not encounter errors when issuing kubectl commands. 

By default, kubectl will read our configuration file from the `/Users/<your_user>/.kube/config` location. Therefore, we need to put it there. 

Our configuration contains the necessary context, access token, server location, amongst other things, so that kubectl knows where to connect to.

Assuming we have downloaded the configuration and it is in our `Downloads` folder, we can move it to the required location by issuing:

```shell
▶ mv ~/Downloads/kubeconfig ~/.kube/config
```

For further information on setting up kubectl, you can reference the following CoreWeave documentation: [Get Started with Kubernetes](https://docs.coreweave.com/coreweave-kubernetes/getting-started)

### Download Stable Diffusion

In this section, we will use the `git` utility to go and acquire the necessary files for setting up Stable Diffusion in our Kubernetes deployment.

Using the `git` utility, we need to clone the following repository locally: [coreweave/kubernetes-cloud](https://github.com/coreweave/kubernetes-cloud)

If `git` is not available locally, we can install it using:

```shell
▶ brew install git 
```

> ℹ️ This respository contains examples for getting started with Kubernetes on the CoreWeave platform


Issue the following command to get started:


```shell
▶ git clone git@github.com:coreweave/kubernetes-cloud.git
```

Navigate to the package files for stable-diffusion:

```shell
▶ cd kubernetes-cloud/online-inference/stable-diffusion
```

---

### Setup kubectl


As part of our kubeconfig, we have a context we will need to switch into, to be able to tell kubectl to target a specific Kubernetes environment.

> ℹ️ A kubeconfig can contain multiple contexts for different environments. We select the correct context to deploy into.

We can switch to the `coreweave` context using:

```shell
▶ kubectl config use-context coreweave
```

> ℹ️ [kubie](https://github.com/sbstp/kubie) or [kubectx](https://formulae.brew.sh/formula/kubectx) are alternative tools that can be used to switch between kubectl contexts which also have additional cool features!

---

### Deploying the Stable Diffusion Service

Within the above repository we should now be inside, we will see a file called: `02-inference-service-yaml`.
This file contains deployment information for the Stable Diffusion service. Consider it a "blueprint" for creating pod(s).

We can understand that this is a service as the file contains:

```yaml
kind: Service
```

We can feed this YAML file into Kubernetes via kubectl, using the following command, to begin deployment:

```shell
▶ kubectl apply -f 02-inference-service.yaml
```

We should then see a message saying it has been created and after some time, since Kubernetes needs to build and scale the pods, we can then verify the service by issuing:

```shell
▶ kubectl get ksvc stable-diffusion
```

If we wanted to view any pods that this deployment has created, we can issue the following commands to get a list of pods:

```shell
▶ kubectl get pods
```

Using the pod information gleaned from the above command, we can go further and see how the pod is built alongside some other technical information that may be of use for future troubleshooting/debugging efforts.

We can issue the following command to "describe" our newly created pods:

```shell
▶ kubectl describe pod sd-00001-deployment-7c7897bb68-ghhhs
```


##### Example outputs:

```shell
▶ kubectl get ksvc stable-diffusion
NAME   URL                                                           LATESTCREATED   LATESTREADY   READY   REASON
sd     https://sd.tenant-36ac33-proj2.knative.ord1.coreweave.cloud   sd-00001        sd-00001      True
```

```shell
▶ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
sd-00001-deployment-7c7897bb68-ghhhs   2/2     Running   0          24h
```

```shell
▶ kubectl describe pod sd-00001-deployment-7c7897bb68-ghhhs
Name:                 sd-00001-deployment-7c7897bb68-ghhhs
Namespace:            tenant-36ac33-proj2
Priority:             1000000
Priority Class Name:  normal
Service Account:      default
Node:                 gbcb9ec/10.135.40.2
Start Time:           Fri, 14 Jun 2024 18:40:43 +0100
...
```

#### Interacting with Stable Diffusion via REST API

To be able to interact with the service, we need to craft a POST request to send to the Stable Diffusion RESTful API.

The POST request will need to include a payload, or "body", that contains a field in it called "prompt", which Stable Diffusion will interpret as context for our image.

The following command can be used to create an image, noting the `--output` pointing to a local filename:

> ℹ️ Please change \<namespace> with your actual namespace


```shell
▶ curl -X 'POST' \
  'https://sd.<namespace>.knative.ord1.coreweave.cloud/generate' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "an IT engineer working hard at their desk",
  "guidance_scale": 7,
  "num_inference_steps": 28,
  "seed": 0,
  "width": 512,
  "height": 512
}' --output it_engineer.png
```

Once this has come back, hopefully without error, you will be able to issue the following command to open the image and view the results:

```shell
▶ open it_engineer.png
```

---

### Known Issues

- The link contained in the Stable Diffusion `README.md` contains a link that 404's, specifically here: [stable-diffusion](https://github.com/coreweave/kubernetes-cloud/tree/master/online-inference/stable-diffusion)
  - This should actually point to: [Stable Diffusion: Text-to-image](https://docs.coreweave.com/coreweave-machine-learning-and-ai/how-to-guides-and-tutorials/examples/hugging-face-guides/pytorch-hugging-face-diffusers-stable-diffusion-text-to-image)


### Appendix

- For more information on Stable Diffusion, please visit: [Stability.ai](https://stability.ai/stable-image)


