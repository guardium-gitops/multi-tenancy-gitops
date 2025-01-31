# Deploy Cloud Pak for Security - Guardium Insights

## Storage

The [current validated storage solution](https://www.ibm.com/docs/en/guardium-insights/3.1.x?topic=requirements-validated-storage-options) for Guardium Insights outside of IBM Cloud is OpenShift Data Foundation.  To use OpenShift Data Foundation as your storage solution for Guardium Insights, first follow the [ODF Recipe](./openshift-container-storage-recipe.md) and then continue with this guide.

## Infrastructure - Kustomization.yaml

1. Edit the Infrastructure layer `${GITOPS_PROFILE}/1-infra/kustomization.yaml` and un-comment the following:

    ```yaml
    - argocd/consolenotification.yaml
    - argocd/namespace-ibm-common-services.yaml
    - argocd/namespace-sealed-secrets.yaml
    - argocd/namespace-tools.yaml
    - argocd/serviceaccounts-tools.yaml
    ```

## Services - Kustomization.yaml

1. Edit the Cloud Pak for Data Platform instance and update the storage class `${GITOPS_PROFILE}/2-services/argocd/instances/ibm-cpd-instance.yaml` as needed.  The default is set to `managed-nfs-storage`.

    ```yaml
      - name: spec.storageClass
        value: "managed-nfs-storage"
    ```

2. Edit the Watson Openscale instance and update the storage class `${GITOPS_PROFILE}/2-services/argocd/instances/ibm-cpd-wos-instance.yaml` as needed.  The default is set to `managed-nfs-storage`.

    ```yaml
      - name: spec.storageClass
        value: managed-nfs-storage
    ```

3. Edit the Services layer `${GITOPS_PROFILE}/2-services/kustomization.yaml` uncomment the following:

    ```yaml
    - argocd/operators/ibm-cpd-scheduling-operator.yaml
    - argocd/operators/ibm-cpd-platform-operator.yaml
    - argocd/instances/ibm-cpd-instance.yaml
    - argocd/operators/ibm-cpd-wos-operator.yaml
    - argocd/instances/ibm-cpd-wos-instance.yaml
    - argocd/operators/ibm-catalogs.yaml
    - argocd/instances/sealed-secrets.yaml
    ```

## Validation

1. Get the status of the control plane (lite-cr)

    ```bash
    oc get ZenService lite-cr -n tools -o jsonpath="{.status.zenStatus}{'\n'}"
    ```

    Cloud Pak for Data control plane is ready when the command returns `Completed`. If the command returns another status, wait for some more time and rerun the command.

2. Get the status of Watson Openscale (analyticsengine-sample)

    ```bash
    oc get WOService aiopenscale -n tools -o jsonpath="{.status} {'\n'}"
    ```

    Watson Openscale is ready when the command returns `Completed`.

3. Get the URL of the Cloud Pak for Data web client and open it in a browser.

    ```bash
    echo https://`oc get ZenService lite-cr -n tools -o jsonpath="{.status.url}{'\n'}"`
    ```

4. The credentials for logging into the Cloud Pak for Data web client are `admin/<password>` where password is stored in a secret.

    ```bash
    oc extract secret/admin-user-details -n tools --keys=initial_admin_password --to=-
    ```
