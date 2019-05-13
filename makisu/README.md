# makisu

This Task builds source into a container image using uber's
[`makisu`](https://github.com/uber/makisu) tool.

>Makisu is a fast and flexible Docker image build tool designed for unprivileged
>containerized environments such as Mesos or Kubernetes.
> - [makisu website](https://github.com/uber/makisu)

## Create the registry configuration

makisu uses a [registry
configuration](https://github.com/uber/makisu/blob/master/docs/REGISTRY.md)
which should be stored as a secret in Kubernetes. Adjust the `registry.yaml` in
this diretroy to contain your user and password for the Docker hub (or
configure a different
[registry](https://github.com/uber/makisu/blob/master/docs/REGISTRY.md#examples)).
Keep in mind that the secret must exist in the same namespace as the build
runs.:

```bash
kubectl --namespace default create secret generic docker-registry-config --from-file=./registry.yaml
```

## Install the Task

```
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/makisu/makisu.yaml
```

## Inputs

### Parameters

* **CONTEXTPATH**: The path to the build context (_default:_
  `/workspace`)
* **PUSH_REGISTRY**: The Registry to push the image to (_default:_
  `index.docker.io`)
* **REGISTRY_SECRET**: Secret containing information about the used regsitry (_default:_
  `docker-registry-config`)

### Resources

* **source**: A `git`-type `PipelineResource` specifying the location of the
  source to build.

## Outputs

### Resources

* **image**: An `image`-type `PipelineResource` specify the image that should be built.

## Usage

This TaskRun runs the Task to fetch a Git repo, and build and push a container
image using makisu.

In this example, the Git repo being built is expected to have a `Dockerfile` at
the root of the repository.

### Docker Registry

```yaml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: example-run
spec:
  taskRef:
    name: makisu
  inputs:
    resources:
    - name: source
      resourceSpec:
        type: git
        params:
        - name: url
          value: https://github.com/my-user/my-repo
  outputs:
    resources:
    - name: image
      resourceSpec:
        type: image
        params:
        - name: url
          value: gcr.io/my-repo/my-image
```

### Other Registries

The `PUSH_REGISTRY` **must** match the name of the registry specified in the registry.yaml

```yaml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: example-run-gcr
spec:
  taskRef:
    name: makisu
  inputs:
    params:
    - name: PUSH_REGISTRY # must match the registry in the secret
      value: eu.gcr.io
    - name: REGISTRY_SECRET
      value: gcr-registry-config
    resources:
    - name: source
      resourceSpec:
        type: git
        params:
        - name: url
          value: https://github.com/my-user/my-repo
  outputs:
    resources:
    - name: image
      resourceSpec:
        type: image
        params:
        - name: url
          value: gcr.io/my-repo/my-image
```