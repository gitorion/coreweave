+++
title = '☸️ My CoreWeave Project Deployment'
date = 2024-06-14T16:10:06+01:00
draft = false
summary = "A post about my journey learning more about CoreWeave." 
+++


## 🚀 Deploying Stable Diffusion API on Kubernetes

The primary task was to deploy a **Stable Diffusion** model that is served via an API.
I used the Kubernetes configuration file (kubeconfig) provided to me alongside the following documentation: [Stable Diffusion: Text to Image](https://docs.coreweave.com/coreweave-machine-learning-and-ai/how-to-guides-and-tutorials/examples/hugging-face-guides/pytorch-hugging-face-diffusers-stable-diffusion-text-to-image).

> ℹ️ I did learn through use of CoreWeave that if my _kubeconfig_ file was misplaced, that I could acquire it again in the CoreWeave WebUI in Namespaces.

Using the kube configuration, I was able to start inspecting what I was capable of doing, as I learned that the token limited me to do only certain actions. 
Following the guide, I was able to successfully deploy the images and create pods that hosted the API for Stable Diffusion. 

I tested the deployment by issuing cURL POST requests with certain text to see if the response would return me an image that was accurate and met the conditions I set in the POST request.
The images returned were accurate. However, what I did notice was that if the description was too verbose, the model began to struggle and produce inaccuracies in the image.

The ability to be able to issue a POST request to generate an image on the fly based on certain text though is very cool! I can see where this could be effective if there is a dynamic need in a script for an image where we can feed the script a POST request to the model and get it to generate a dynamic image. For instance, if the script was to say hello to you, but we needed a name from you, the script could ask for your name and return an image with the name while saying hello. 


### ✉️ POST Request Examples

The 3 images I generated from the model are below, coupled with their corresponding POST request:
```shell
curl -X 'POST' \
  'https://sd.tenant-36ac33-proj2.knative.ord1.coreweave.cloud/generate' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "a beautiful beach at sunset",
  "guidance_scale": 7,
  "num_inference_steps": 28,
  "seed": 0,
  "width": 512,
  "height": 512
}' --output image1.png
```
![a beautiful beach at sunset](/images/image1.png "A beautiful beach!")

```shell
curl -X 'POST' \
  'https://sd.tenant-36ac33-proj2.knative.ord1.coreweave.cloud/generate' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "a clear night sky",
  "guidance_scale": 7,
  "num_inference_steps": 28,
  "seed": 0,
  "width": 512,
  "height": 512
}' --output image2.png
```
![a clear night sky](/images/image2.png "A Clear Night Sky!")

```shell
curl -X 'POST' \
  'https://sd.tenant-36ac33-proj2.knative.ord1.coreweave.cloud/generate' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "a large scottish style castle at night",
  "guidance_scale": 7,
  "num_inference_steps": 28,
  "seed": 0,
  "width": 512,
  "height": 512
}' --output image3.png
```
![a large scottish style castle at night](/images/image3.png "A Scottish Castle!")

### Kubernetes Learnings

I began looking more into how Kubernetes works and just getting to grips with the basics. The CoreWeave documentation helped me understand how deployments work.

#### Deployments

- Kubernetes deployments are deployed using YAML files.
  - These deployments can be also called "Manifests".

- The manifest contains instructions to tell Kubernetes how to build something.
  - This could be different _kind_'s of things, such as `Service`, `Config`, `Secret`, `Deployment`. (In the case of this task, we were deploying a `Service`)

- The manifest will also contain information such as:
  - Name
  - Image
  - Resources
    - Cores
    - Memory
  - Environment variables
  - Networking
    - Ports
  - Replicas

  Amongst other things.

The manifest itself is an interpreted script of instructions we give Kubernetes to deploy something in a given way. This could also be viewed as our deployment's entry point into Kubernetes.

#### 👨‍🚀 Lift off!

Once a deployment has been applied and Kubernetes has built the pods, we can actually go and get further information on how the pod has been deployed within the namespace:

```shell
▶ kubectl describe pod sd-00001-deployment-7c7897bb68-ghhhs
```

I can see the values generated for this deployment inside the created Kubernetes pod.  

From a configuration and troubleshooting perspective, a few values immediately jump out worthy of note:

```shell
Name:           tenant-36ac33-proj2
Namespace:      tenant-36ac33-proj2
Node:           gbcb9ec/10.135.40.2
Status:         Running
Started:        Fri, 14 Jun 2024 18:40:46 +0100
Controlled By:  ReplicaSet/sd-00001-deployment-7c7897bb68
```

There are certain things that I am unable to see though as the role that I have does not have access to see certain resources, such as Secrets.

I suppose this is due to the RBAC Token provided to me which details my role and what I can and cannot do. More information on RBAC tokens are found here: [Custom RBAC Access Tokens](https://docs.coreweave.com/coreweave-kubernetes/getting-started/custom-rbac-access-tokens)

> ℹ️ Information about the deployment, such as resources used can also be viewed in the CoreWeave WebUI under Namespaces!

#### 💭 Thoughts

As an individual who has not used Kubernetes extensively at a professional level it has been a lot of fun learning and exploring the power and possibilities of the solution.

It reminds me of the first time I was able to use Docker and to be able to understand how containers worked.
This has let me consider further about orchestration of containers and the deployment of reliable infrastructure.

My next step in learning Kubernetes will be to follow the guide by Kelsey Hightower: [Kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) so that I can gain more in-depth knowledge on Kubernetes.

