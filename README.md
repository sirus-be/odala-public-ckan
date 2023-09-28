# Components - CKAN
This CKAN solution uses a custom build CKAN container image and a helm chart to deploy the custom CKAN container and other services (solr, datapusher,...)
The container image should be present in the GitLabs docker registry.
In this document the build and deployment of the ckan image is explained

## License
for the ODALA project.

© 2023 Sirus

License EUPL 1.2

![](https://ec.europa.eu/inea/sites/default/files/ceflogos/en_horizontal_cef_logo_2.png)

## CKAN Build and deploy the custom Ckan-Odala image to docker regsitry
To build, tag, and push docker images to the private docker registry or other docker registry:
1.  Go into the root of the CKAN directory (top level)
2.  Login to the docker registry using  
    `docker login gitlab.publiccode.solutions:5050` or other docker registry
3.  Build the images    
    `docker build -t gitlab.publiccode.solutions:5050/odala/components_ckan/ckan-odala:1.0.10 ./ckan-odala` 
5. Push the image to the store 
    `docker push gitlab.publiccode.solutions:5050/odala/components_ckan/ckan-odala:1.0.10`

## Release: gitlab pipelines
Add the image to the registry (previous steps)
Change the ckan-odala image version number in values.yaml
The release process is automated if you use the cicd pipepeline: .gitlab-ci.yml, the pipeline is triggered when changes are pushed to a branch with prefix odala-staging

## Ingress (not part of the pipeline)
kubectl apply -f ingress.yaml --namespace odala-staging
 
## Release issues
- If the ckan container is in failed state (mostly after new release): delete manually and a new one will spin up
- Custom property pvc.createckanpvc to prevent recreation of volume (dataloss, e.g. site images) when deploying. Set to true to recreate volume (needs to be deleted on the cluster first)
- https://gitlab.publiccode.solutions/odala/components_ckan/-/issues/12: The existing datasets "disappeared": you can look them up via Rest Api or hardcoded in Document: Datasets.MD, open the dataset -> Groups  -> Remove Group -> Add the Group again. 
Cause: https://github.com/ckan/ckan/issues/4425 https://stackoverflow.com/questions/57141194/no-datasets-found-after-stopping-starting-ckan-docker-containers
The solr volume is removed on every deploy.
Solution: let solr run / see github solutions / working with helm upgrade, this will work as long as solr isn't changed in the helm chart

### Create pull secret for the docker registry on kubernetes cluster
- In GitLab
https://docs.gitlab.com/ee/user/project/deploy_tokens/#creating-a-deploy-token
- As Kubernetes secret
```kubectl create secret docker-registry gitlab-regcred --docker-server=gitlab.publiccode.solutions:5050 --docker-username=gitlab+deploy-token-1 --docker-password=PWHERE```

## Add ckan extensions
`https://github.com/keitaroinc/ckan-helm/issues/23`

## licenses
- add license to ckan-odala/licenses/licenses.json
- Build custom docker image (ckan-odala/Dockerfile):
    - Add the file: ADD licenses/licenses.json licenses.json
    - Set config: config-tool ${APP_DIR}/production.ini "licenses_group_url = file://licenses.json"

## open api
https://github.com/bcgov/ckanext-openapiviewer
- create vocabulary: resource_format ("name": "resource_format")
- get resource_format id
## https://github.com/bcgov/ckanext-openapiviewer

## DCAT extension
is included in Dockerfile

# CKAN Local
## Cluster
[Kubernetes via Docker-Desktop](https://docs.docker.com/desktop/kubernetes/)

## Nginx
```helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace```

## ingress issues
Make sure that the ingress controller has external ip 'localhost'
```kubectl get service -n ingress-nginx```
issues multiple delete/deploy: external ip status reamains 'pending' (port 80 in use): restart docker-desktop kubernetes  and machine 

## Build image
```docker build -t ckan-odala:1.0.0 .```

## Deploy
```helm install ckan-odala ckan-helm -f ckan-helm/valueslocal.yaml --namespace ckan --create-namespace```
```kubectl apply -f ingresslocal.yaml --namespace ckan```

Access CKAN via 'localhost'

# Cleanup 
```helm delete ckan-odala | 
kubectl delete pvc -l release=ckan-odala | 
kubectl delete pvc solr-pvc-solr-0 | 
kubectl delete pvc data-postgres-0```

If volume ckan needs to be recreated:
```kubectl delete pvc ckan``` 
and enable pvc.createckanpvc in helm values.yml for the next deploy
