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

In this blog post I will explain a relatively straight forward way to get ephemeral runners on Openshift Kubernetes container platform.

### Deployment Assumptions

The main components that need to be deployed are as follows:

1. Cloud or Self Managed [Gitlab](https://gitlab.com/) CICD solution deployment or as a [Kubernetes operator](https://docs.gitlab.com/operator/). In my case I am running the Gitlab linux package in an [Alma Linux](https://almalinux.org/) virtual machine running on a [hypervisor](https://www.proxmox.com/en/).

> Gitlab CICD can also be deployed on an Openshift cluster or any other Kubernetes distribution through a [Helm chart](https://docs.gitlab.com/charts/installation/)

* [Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) cluster or any other Kubernetes engine such as [EKS](https://aws.amazon.com/eks/) or [GKE](https://cloud.google.com/kubernetes-engine). For this demonstration, I have an [Openshift Installation](https://docs.openshift.com/container-platform/4.15/installing/overview/index.html).

> A [single node Openshift install](https://docs.openshift.com/container-platform/4.15/installing/installing_sno/install-sno-installing-sno.html) will also work adequately.

* For the container image builds I will be using [buildah](https://buildah.io/) since it provides the capability to generate OCI container images.

* For local container image build and testing, I would also encourage the use of [podman](https://podman.io/)

* You also need a container registry, for my deployment I have chosen to enable the in-built [Gitlab Container registry](https://docs.gitlab.com/ee/user/packages/container_registry/)

> Technically, it makes more sense to have a seperate container registry solution such as [harbor](https://goharbor.io/docs/2.12.0/install-config/) or [quay](https://docs.projectquay.io/deploy_red_hat_quay_operator.html).    

### Self-Managed Gitlab CICD and Runners

Before we go further, I will provide a brief overview of Gitlab. A code repository is setup as part of a [Gitlab project](https://docs.gitlab.com/ee/user/project/). Projects can be setup at the user or group level.

[Gitlab Runners](https://docs.gitlab.com/runner/). Since this is a self-managed deployment, we have to provide runners for the CICD server to execute devops pipelines and jobs.

Runners can run on VMs but it is makes more sense to have a containerised runner infrastructure since they are ephemeral in nature.

### Runner Deployment on Openshift or Kubernetes

You have a couple of options in deplying the runner infrastructure on openshift. 

It can either be deployed as a [helm chart](https://docs.gitlab.com/runner/install/kubernetes.html) or as an [Openshift operator](https://docs.gitlab.com/runner/install/operator.html). 

In my case, I have installed the operator from the [openshift marketplace](https://docs.openshift.com/container-platform/4.15/applications/red-hat-marketplace.html). 

It only takes a few clicks to install the operator into Openshift and we are good to go!

[![Operators](/assets/blog2-a-operators-installed.png "Operators")]()

#### Runner configuration on Gitlab

You need to setup runners at the group or project level. It makes sense to have runners setup at the group level. You can also have multiple runner configuration that can be made available to your end users if required.

[![Runners](/assets/blog2-a-runners-configured.png "Runners")]()

In my case, I have multiple runners configured.

* One is a buildah runner with the runner image from [quay.io](https://quay.io/buildah/stable).

* Additional runners have been configured using the [Redhat UBI 9 Micro and Minimal Images](https://developers.redhat.com/articles/ubi-faq#).

##### Let's look at the Runner Manifests

Once the runners are configured instance. You need to create runner instance in Kubernetes.

For configuring a runner instance the following information is critical:

1. Gitlab instance URL coordinates
1. Kubernetes secret with the gitlab runner token. the token associates the runner deployment with the Gitlab runner instance.

Optionally, you can pass the following to the configuration data:

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

In case of any issues refer to the official documenation for configuring the [runner operator on Gitlab](https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html)

You can also refer to the official documentation on setting up [ rootless buildah runners on Gitlab](https://docs.gitlab.com/ee/ci/docker/buildah_rootless_tutorial.html)

#### A Sample Pipeline Manifest snippet

A Gitlab pipeline is best configured as a yaml manifest. A pipeline will essentially contain a series of jobs. Jobs are best organised into various stages.

[![Pipeline](/assets/blog2-a-pipeline-run-success.png "Pipeline")]()

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

#### Runner operation in progress

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

## Important considerations

### Gitlab CICD and Container Registry

* The community edition is quite feature rich but requires considerable compute and storage resources.
* Considerable technical debt will be incurred if you choose to use Self signed certificates when setting up your Gitlab server and container registry.
* Gitlab supports [Let's Encrypt](https://letsencrypt.org/) certificates and you can use the [certbot](https://certbot.eff.org/) tool to create certificates for your deployment.

### Kubernetes and Openshift

* I have not deployed Kubernetes storage in this demo setup and I have gotten away with `emptyDir`. But a more robust deployment could be configured with persistent volume claims.

For further information refer to [Configuring Storage for Runners](https://docs.gitlab.com/runner/executors/kubernetes/#configure-volume-types)

* It is best to run un-privileged containers in Openshift, but you can provide increase permissions by passing higher privilege secuirty context to the runner service account with the role base access control mechanism.

* If you setup runners through the marketplace community operator, there may be cases when the gitlab runner operator is not available. This can occur if you are on the latest versions of Openshift.

## Troubleshooting Section

* In some cases pipeline jobs can fail due to insufficient resources or other technical glitches.  

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

> here you can review the log to validate if the correct runner image is being used for your job 

* If you encounter such issues you can put retry logic in your Gitlab pipeline manifest.

## Closing thoughts

This setup will help you to achieve the following outcomes:

1. Setup a scalable ephemeral runner infrastructure on Kubernetes.
1. Established mutliple types of runners to suit the pipeline requirements
1. Leveraged `Tags` to select the appropriate runner for the job.