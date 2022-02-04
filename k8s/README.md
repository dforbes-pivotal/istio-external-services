# Kubernetes Resources

Deploy these resources to your current Kubernetes context by:

1. Ensuring that your current working directory is this `k8s` directory
1. Applying all manifests:

    ```sh
    kubectl apply -f .
    ```

    See the manifests in this directory for all of the resources that will be created.

To see the status of all resources created for this demo, run:

```sh
kubectl get all,cm,gw,se,vs,dr -l demo=istio-external-services
```

To clean up afterwards:

1. Either

    ```sh
    kubectl delete -f .
    ```

    or

    ```sh
    kubectl delete all,cm,gw,se,vs,dr -l demo=istio-external-services -A
    ```
