# TL;DR

This lab will build ones understanding and skills using remote images. You will learn how the different build strategies use remote images and how to leverage artifactory as a pull-through-cache to improve build times and reduce load on the bcgov network infrastructure.

# Using Remote Images in a Build

### Prerequisite(s)

On our OCP4 clusters and going forward you are automatically provisioned Artifactory credentials when your `tools` namespace is created. 

To find them, list your secrets and pick out the one with `artifactory` in the name:

You'll find one with the name:

```console
oc get secrets
```

Results in

```console
➜  openshift-workshop git:(master) ✗ oc get secrets
NAME                                 TYPE                                  DATA   AGE
artifactory-serviceaccount-default   kubernetes.io/basic-auth              2      6d22h
builder-dockercfg-vv8ft              kubernetes.io/dockercfg               1      6d22h
builder-token-jwr22                  kubernetes.io/service-account-token   4      6d22h
builder-token-zg7jv                  kubernetes.io/service-account-token   4      6d22h
default-dockercfg-ct5jj              kubernetes.io/dockercfg               1      6d22h
default-token-8fz6m                  kubernetes.io/service-account-token   4      6d22h
default-token-vt8kh                  kubernetes.io/service-account-token   4      6d22h
deployer-dockercfg-7c6vq             kubernetes.io/dockercfg               1      6d22h
deployer-token-gn7qs                 kubernetes.io/service-account-token   4      6d22h
deployer-token-q82hr                 kubernetes.io/service-account-token   4      6d22h
```

- Create a `docker` secret for pulling images
- Setting up NSP so that build images can reach outside of the cluster; in the case of S2I builds this is needed to fetch the source code.

### Docker Strategy

![Docker Strategy](./doc/docker-strategy.png "Docker Strategy")

### S2I Strategy

![S2I Strategy](./doc/s2i-strategy.png "S2I Strategy")

##
10666  export AFPWD=$(oc get secret/artifactory-serviceaccount-default -o json | jq '.data.password' |  tr -d "\"" | base64 -d)
10667  export AFUSR=$(oc get secret/artifactory-serviceaccount-default -o json | jq '.data.username' |  tr -d "\"" | base64 -d)

oc create secret docker-registry artifactory-dockercfg --docker-server=docker-remote.artifacts.developer.gov.bc.ca --docker-username=$AFUSR --docker-password=$AFPWD --docker-email=unused

oc get secret/artifactory-dockercfg -o json | jq '.data.".dockerconfigjson"' |tr -d "\"" | base64 -d

oc secrets link builder artifactory-dockercfg --for=pull
oc secrets link default artifactory-dockercfg --for=pull

now update docker.io to docker-remote.artifacts.developer.gov.bc.ca

```yaml
    strategy:
      dockerStrategy:
        pullSecret:
          name: artifactory-dockercfg
```


For a `type: Docker` build, you need either:
Line 32-33 OR Line 34-36 **with** the command:

```yaml
    source:
      type: Dockerfile
      dockerfile: |-
        FROM docker-remote.artifacts.developer.gov.bc.ca/node:lts-alpine
        RUN echo "Hello!"
    strategy:
      dockerStrategy:
        pullSecret:
          name: artifactory-dockercfg
        from: 
          kind: DockerImage
          name: docker-remote.artifacts.developer.gov.bc.ca/node:lts-alpine
      type: Docker
```
Also, just making notes here :) Skipping A and B will fail with or without the command.

```console
oc secrets link builder artifactory-dockercfg --for=pull,mount
```
or
```console
oc secrets link builder artifactory-dockercfg
```

For a `type: Source` you always need 55-57; line 53-54 without the command or
skip like 53-54 and use the command.

```yaml
      sourceStrategy:
        # pullSecret:
        #   name: artifactory-dockercfg
        from: 
          kind: DockerImage
          name: docker-remote.artifacts.developer.gov.bc.ca/fullboar/nginx-118
```