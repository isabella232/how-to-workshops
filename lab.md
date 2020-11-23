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