---
author: Nishad Saithaly
pubDatetime: 2025-02-03T07:17:07Z
title: Gitlab cicd runners on Openshift Kubernetes
postSlug: gitlab-cicd-runners-openshift-k8s
featured: true
draft: false
tags:
  - gitlab
  - openshift
  - runner
  - cicd
  - devops
  - kubernetes
ogImage: "https://raw.githubusercontent.com/mailman-2097/nishad.link/master/public/assets/blog2.svg"
description: Setting up Gitlab CICD with container runners on Openshift Kubernetes
---

## Table of contents

## Introduction

[![CICD](/assets/blog2.png "CICD")]()

In this blog post I shall explain a relatively straight forward way to get ephemeral runners on Openshift Kubernetes container platform.

## Solution Deployment

The main components that need to be deployed are as follows:

__Gitlab Server__

1. Cloud or Self Managed [Gitlab](https://gitlab.com/) CICD solution deployment or as a [Kubernetes operator](https://docs.gitlab.com/operator/) which is an easy install method in Openshift. 

In my case, I have chosen the linux installation method for setting up Gitlab.
I am using an [Alma Linux](https://almalinux.org/) virtual machine running on a [hypervisor](https://www.proxmox.com/en/).

> Gitlab CICD can also be deployed on an Openshift cluster operator or any other Kubernetes distribution through a [Helm chart](https://docs.gitlab.com/charts/installation/)

__Kubernetes Cluster__

* [Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) cluster or any other Kubernetes engine such as [EKS](https://aws.amazon.com/eks/) or [GKE](https://cloud.google.com/kubernetes-engine). For this demonstration, I have an [Openshift Installation](https://docs.openshift.com/container-platform/4.15/installing/overview/index.html).

> A [single node Openshift install](https://docs.openshift.com/container-platform/4.15/installing/installing_sno/install-sno-installing-sno.html) will also work adequately.

__Container Registry__

* You also need a container registry, for my deployment I have chosen to enable the in-built [Gitlab Container registry](https://docs.gitlab.com/ee/user/packages/container_registry/)

> Technically, it makes more sense to have a seperate container registry solution such as [harbor](https://goharbor.io/docs/2.12.0/install-config/) or [quay](https://docs.projectquay.io/deploy_red_hat_quay_operator.html).    

__Container Tooling__

* For the container image builds I will be using [buildah](https://buildah.io/) since it provides the capability to generate OCI container images.

* For local container image build and testing, I would also encourage the use of [podman](https://podman.io/)

### Self-Managed Gitlab CICD and Runners

Before we go further, I will provide a brief overview of Gitlab logical structures. 

A code repository is setup as part of a [Gitlab project](https://docs.gitlab.com/ee/user/project/). Projects can be setup at the user or group level.

[Gitlab Runners](https://docs.gitlab.com/runner/) will need to be setup as part of a self-managed deployment. The runners once configured successfully can execute your devops pipelines and jobs.

Runner agents can also be run on Virtal Machines but it is makes more sense to have a containerised runner infrastructure since they are ephemeral in nature and provide better scalability.

### Runner Deployment on Openshift or Kubernetes

You have a couple of options in deplying the runner infrastructure on openshift. 

It can either be deployed as a [helm chart](https://docs.gitlab.com/runner/install/kubernetes.html) _or_ as an [Openshift operator](https://docs.gitlab.com/runner/install/operator.html). 

b. In my case, I have installed the operator from the [openshift marketplace](https://docs.openshift.com/container-platform/4.15/applications/red-hat-marketplace.html). 

It only takes a few clicks to install the operator into Openshift and we are good to go! So I chose this method.

[![Operators](/assets/blog2-a-operators-installed.png "Operators")]()

#### Runner configuration on Gitlab

You need to setup runners at the group or project level. To me it made more sense to have runners setup at the group level. You can and ideally should have multiple runner configurations that can be made available to your end users as required.

[![Runners](/assets/blog2-a-runners-configured.png "Runners")]()

In my case, I have multiple runners configured:

* One is a buildah runner with the runner image from [quay.io](https://quay.io/buildah/stable).

* Additional runners have been configured using the [Redhat UBI 9 Micro and Minimal Images](https://developers.redhat.com/articles/ubi-faq#).

##### Let's look at the Runner Instance Manifests

Once the runners are configured instance in Gitlab. 

You need to deploy a runner in Kubernetes.

For configuring a runner instance in Kubernetes, the following information is critical:

1. Gitlab instance URL
1. Kubernetes secret with the gitlab runner token. The token associates the runner deployment with the Gitlab runner instance and is generated from the Gitlab Server.

Optionally, you can pass additional configuration data to suit your particular needs:

* Custom Runner Configuration [custom-config.toml](https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html#customize-configtoml-with-a-configuration-template) required incase you need to configure your runner

* Override runner [environment variables](https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html#configure-a-proxy-environment)

```yaml

apiVersion: apps.gitlab.com/v1beta2
kind: Runner
metadata:
  name: gitlab-runner-buildah
  namespace: gitlab-runner
spec:
  gitlabUrl: https://gitlab.homelab0.nishad.link
  buildImage: quay.io/buildah/stable:latest
  token: gitlab-runner-buildah-secret
  config: gitlab-runner-buildah-config
  env: gitlab-runner-buildah-env

```

In case of any issues, refer to the official documenation for configuring the [runner operator on Gitlab](https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html)

You can also refer to the official documentation on setting up [ rootless buildah runners on Gitlab](https://docs.gitlab.com/ee/ci/docker/buildah_rootless_tutorial.html)


If all goes to plan, as it occassionally does 😜. 

🎉 All your runners will be online in Gitlab and ready for use. 🎉

[![Runners Online](/assets/blog2-a-runners-online-in-gitlab.png "Runners Online")]()

#### A Sample Pipeline Manifest snippet

A Gitlab pipeline is best configured as a yaml manifest. A pipeline will essentially contain a series of jobs. Jobs are best organised into various stages.

```yaml

image-build:
  stage: build-n-push
  tags:
    - linux, shared, ocp
  script: |
    buildah --version
    buildah bud $BUILD_EXTRA_ARGS \
              --tls-verify="$TLSVERIFY" \
              --layers \
              -f "$DOCKERFILE" \
              -t "$IMAGE" \
              "$BUILD_CONTEXT"
    echo "$CI_REGISTRY_PASSWORD" | buildah login --username "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"
    buildah push $PUSH_EXTRA_ARGS \
              --tls-verify="$TLSVERIFY" \
              --digestfile /tmp/image-digest $IMAGE docker://$IMAGE:$COMMIT_TAG

only:
    - /^feat\/.*/
    - /^bugfix\/.*/
    - main
    - master

micro-runner-test:
  stage: validate
  tags:
    - ubi9micro, shared, ocp
  script: |
    echo "This will run in ubi 9 micro runner"
  only:
    - /^feat\/.*/
    - /^bugfix\/.*/
    - main
    - master

```
* You may need to configure the buildah runner by passing various configuration options. You can refer to the buildah documentation.

* `Tags` are an important way to select the appropriate runner for your pipeline or job. Make sure to setup the correct runner `tags` in Gitlab such that they can be utilised in your pipeline

#### A successful pipeline execution

[![Pipeline](/assets/blog2-a-pipeline-run-success.png "Pipeline")]()

## Important considerations 

🚨 Before you embark on your journey, you need to review the following details. 🚨

### Gitlab CICD and Container Registry

* The community edition is quite feature rich thus requires considerable compute and storage resources.
* *Considerable* technical debt will be incurred if you choose to use Self signed certificates when setting up your Gitlab server and container registry.
* Gitlab supports [Let's Encrypt](https://letsencrypt.org/) certificates and you can use the [certbot](https://certbot.eff.org/) tool to create certificates for your deployment.

### Kubernetes and Openshift

* I have not deployed Kubernetes storage as part of this demonstration but I have gotten away with using the `emptyDir` storage. But a more robust deployment should have been configured with persistent volume claims.

For further information refer to [Configuring Storage for Runners](https://docs.gitlab.com/runner/executors/kubernetes/#configure-volume-types)

* It is best to run un-privileged containers in Openshift, but if you need to run the pods with higher privileges you can do so by associating a higher privilege security context to the gitlab runner service account using role base access controls.

* If you setup runners through the marketplace community operator, there may be cases when the gitlab runner operator is not available. This can occur if you are on the latest version of Openshift and the community operator has not been published for that version.

## Troubleshooting 

The gitlab console provides a debug console where you can review the pipeline logs to validate if the correct runner image is being used for your job and conduct other troubleshooting.

### Insufficient resource or other scheduling issues

* In some cases, pipeline jobs can fail due to insufficient resources or other technical glitches.  

```bash
Running with gitlab-runner 17.7.0 (3153ccc6)
  on gitlab-runner-ubi9micro-runner-698bbff49b-p5l6b t2_MVTdfZ, system ID: r_ECbstx8CpvGu
Preparing the "kubernetes" executor 00:00
Using Kubernetes namespace: gitlab-runner
Using Kubernetes executor with image registry.access.redhat.com/ubi9-micro:latest ...
Using attach strategy to execute scripts...
Preparing environment 03:06
Using FF_USE_POD_ACTIVE_DEADLINE_SECONDS, the Pod activeDeadlineSeconds will be set to the job timeout: 1h0m0s...
Waiting for pod gitlab-runner/runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh to be running, status is Pending
	Unschedulable: "0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.."
Waiting for pod gitlab-runner/runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh to be running, status is Pending
	Unschedulable: "0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.."
```

* If you encounter such issues you can put retry logic in your Gitlab pipeline manifest.

### Monitoring your runner pods

You can monitor the runners from the command line as the pipeline jobs run.

```bash
[nishad@WSL-RH9A-BYD ocp-runner]$ oc get pods -n gitlab-runner --watch
NAME                                                READY   STATUS    RESTARTS   AGE
gitlab-runner-buildah-runner-86d7cd9b8b-btrkn       1/1     Running   0          36m
gitlab-runner-ubi9micro-runner-698bbff49b-p5l6b     1/1     Running   0          36m
gitlab-runner-ubi9minimal-runner-7fdcbc9898-4f7x8   1/1     Running   0          28m
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     2/2     Running   0          2m1s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Pending   0          112s
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     2/2     Terminating   0          2m43s
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     0/2     Terminating   0          3m13s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Pending       0          3m4s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Pending       0          3m4s
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     0/2     Terminating   0          3m13s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Init:0/1      0          3m4s
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     0/2     Terminating   0          3m13s
runner-t1tbpaxq-project-2-concurrent-0-9r5jiajz     0/2     Terminating   0          3m13s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Init:0/1      0          3m5s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     PodInitializing   0          3m5s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Terminating       0          3m6s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     2/2     Terminating       0          3m7s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Pending           0          0s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Pending           0          0s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Terminating       0          3m37s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Pending           0          7s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Pending           0          7s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Init:0/1          0          8s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     Init:0/1          0          8s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Terminating       0          3m38s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Terminating       0          3m38s
runner-t2mvtdfz-project-2-concurrent-0-7t40yuqh     0/2     Terminating       0          3m38s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     0/2     PodInitializing   0          9s
runner-t2mvtdfz-project-2-concurrent-0-nnja40x0     2/2     Running           0          11
```

## Closing thoughts

This setup should help you to achieve the following outcomes:

1. Setup a scalable ephemeral runner infrastructure on Kubernetes.
1. Established mutliple types of runners to suit the pipeline requirements.
1. Leveraged `Tags` to select the appropriate runner for the job.
1. A robust self-hosted version control and devops solution.
