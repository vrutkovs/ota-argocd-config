# OTA pipeline configuration

### Installation procedure

This depends on Pipelines 1.5 so minimum OCP 4.8 cluster is required.

Install OpenShift GitOps and Pipelines:
```
oc apply -f olm-subscription-gitops.yaml
oc apply -f olm-subscription-pipelines.yaml
```

Wait for pods to appear:
```
echo "Still waiting" ; until [[ $(oc get pods -n openshift-gitops -o name  | wc -l) -ne 0 ]]; do echo -n "."; sleep 3; done; echo ""
```

Bootstrap the cluster:
```
oc apply -f bootstrap/
```

The manifests there would do the following:
* `gitops-permissions.yaml` - allow ArgoCD manage the whole cluster
* `argocd-settings.yaml` - ArgoCD customization:
  * use Openshift OAuth
  * give cluster-admins permissions to manage apps
  * allow route controller set hosts in Routes
  * ignore objects created by Pipelines
* `application-sealed-secrets.yaml` - deploy Bitnami's Sealed Secrets controller from `./sealed-secrets`
* `application-cluster-config.yaml` - apply cluster config from `./apps/argocd`

### Sealed Secrets

See https://github.com/bitnami-labs/sealed-secrets for more information. In short, instead of using
plain k8s secrets, we commit `SealedSecrets` object with secret contents encrypted. The controller notices the CR, decrypts it and creates "Secret" object in the target namespace with decrypted contents.

By default the controller would autogenerate the keys, in order to have a reusable installation, `./secrets/credentials-sealed-secret.yaml` needs to be applied to the cluster.

### ArgoCD configuration

`./apps/argocd` deploys several OTA pipeline projects, grouped as "ota" ArgoCD project. It also sets up User Workload Monitoring, AlertManager config and encrypted secrets for S3, Hydra and repo access.

### TODO:

* Make ArgoCD manage Pipelines subscription controlled by ArgoCD
