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