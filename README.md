# llama-stack-operator
This repo hosts a kubernetes operator that is responsible for creating and managing [llama-stack](https://github.com/meta-llama/llama-stack) server.

### Table of contents

- [Developer Guide](#developer-guide)
    - [Pre-requisites](#pre-requisites)
    - [Build Image](#build-image)
    - [Deployment](#deployment)
- [Deploying Llama Stack Server](#deploying-the-llama-stack-server)
 

## Developer Guide

#### Pre-requisites

- Go version **go1.21**
- operator-sdk version can be updated to **v1.31.1**


#### Build Image

- Custom operator image can be built using your local repository

  ```commandline
  make image IMG=quay.io/<username>/llama-stack-k8s-operator:<custom-tag>
  ```

  The default image used is `quay.io/opendatahub/llama-stack-k8s-operator:latest` when not supply argument for `make image`


- Once the image is created, the operator can be deployed either directly, or through OLM. For each deployment method a
  kubeconfig should be exported

  ```commandline
  export KUBECONFIG=<path to kubeconfig>
  ```

#### Deployment

**Deploying operator locally**

- Deploy the created image in your cluster using following command:

  ```commandline
  make deploy IMG=quay.io/<username>/llama-stack-k8s-operator:<custom-tag> 
  ```

- To remove resources created during installation use:

  ```commandline
  make undeploy
  ```

### Deploying the Llama Stack Server

1. Deploy Inference provider server (ollama, vllm etc)
2. Create LlamaStackDistribution CR to get the server running. Example-
```
apiVersion: llama.x-k8s.io/v1alpha1
kind: LlamaStackaDistribution
metadata:
  name: llamastackdistribution-sample
  namespace: <user-defined-namespace>
spec:
  replicas: 1
  server:
    containerSpec:
      image: "llamastack/distribution-ollama:latest"
      port: 8321
      env:
      - name: INFERENCE_MODEL
        value: "meta-llama/Llama-3.2-3B-Instruct" 
      - name: OLLAMA_URL
        value: "http://ollama-server-service.default.svc.cluster.local:11434"
    podOverrides:
      volumes:
      - name: llama-storage
        emptyDir: {}
      volumeMounts:
      - name: llama-storage
        mountPath: "/root/.llama"
``` 
3. Verify the server pod is running in the user define namespace.
