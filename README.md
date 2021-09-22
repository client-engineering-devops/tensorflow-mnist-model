# tensorflow-mnist-model

The `preprocessing.py` and `train.py` processes and trains a mnist model using the quay.io/ibmtechgarage/tensorflow-model-train image. 

### preprocessing.py

The preprocessing script takes in a single argument, `data_dir`. It downloads and preprocesses the data and saves the `pickled` version in `data_dir`. In production code, this would probably be the directory where TFRecords are stored, for example.

## Testing

### locally with docker
```
docker run --name tensorflow-mnist -v containers:/var/lib/containers --rm -it quay.io/ibmtechgarage/tensorflow-model-train
```
Copy Files into container
```
docker cp environment.yml tensorflow-mnist:/tmp/ 
docker cp preprocessing.py tensorflow-mnist:/tmp/ 
docker cp train.py tensorflow-mnist:/tmp/ 
docker cp constants.py tensorflow-mnist:/tmp/ 
docker cp build-script.sh tensorflow-mnist:/tmp/
```
Exec build-script
```
docker exec tensorflow-mnist sh -c ./build-script.sh
```


### tekton pipeline with OpenShift

#### dependencies
image: quay.io/ibmtechgarage/tensorflow-model-train 

#### prerequisite
     
##### Github
- github account
- configure a token that has `admin:repo_hook, repo` 
    - [Generate a GitHub personal access token](https://www.ibm.com/docs/en/cloud-paks/cp-applications/4.2?topic=accelerators-building-deploying-applications)
``` 
# create githubaccount variable
read githubaccount
# create githubtoken variable
read -s githubtoken
```
- fork [tensorflow-mnist-model](https://github.com/itg-devops/tensorflow-mnist-model.git) 


##### Tekton
```
tkn version  
Client version: 0.20.0
Pipeline version: v0.22.0
Triggers version: v0.12.1
```
How to install tkn [cli](https://github.com/tektoncd/cli) 


##### OpenShift Cluster
this is known to work on version: 4.7.23

```
oc version
Client Version: 4.7.13
Server Version: 4.7.23
Kubernetes Version: v1.20.0+558d959
```
How to install oc [cli](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html#cli-installing-cli_cli-developer-commands)

##### Cloud-Native Toolkit
How to install [Cloud-Native Toolkit](https://cloudnativetoolkit.dev)
```
curl -sfL get.cloudnativetoolkit.dev | sh -
```

##### Repos
```
git clone "https://github.com/${githubaccount}/tensorflow-mnist-model.git"
git clone "https://github.com/itg-devops/ibm-garage-tekton-tasks.git"
```

#### Apply the pipeline and tasks to tools project
```
cd ibm-garage-tekton-tasks
git checkout model-pipeline
oc status #check that what project you are in 
oc project tools

oc apply -f tasks/12-train-tensorflow-model-push.yaml
oc apply -f pipelines/tensorflow-model-pipeline.yaml
```

#### Sync and pipeline a `tensorflow-model` 
```
cd ../image-tensorflow-model-train
git status
oc sync tensorflow-model --dev
oc adm policy add-scc-to-user privileged  -z pipeline 
echo $githubtoken | pbcopy
oc pipeline --tekton --pipeline tensorflow-model --param scan-image=false lint-dockerfile=false  health-endpoint=/ 
tkn pr logs -f <pipelinerun>
curl <artfactory url> -O
tar -tvf mnist.tar.gz
```
