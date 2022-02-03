# Example Services on Cloud Foundry

These services are provided for convenience as they will typically be secured with TLS when running
on a Cloud Foundry foundation. Any other secure website or web service could be used, however.

To deploy these services via the terminal, simply:

1. Make sure your working directory is this `cf` directory
1. Log in to your foundation with the `cf` CLI and target the org and space you wish to use, e.g.:  

    ```sh
    cf login -a <CF API endpoint> -o <org> -s <space>
    ```

1. Push both apps:

    ```sh
    cf push -p hello-v1 -f hello-v1/manifest.yml
    cf push -p hello-v2 -f hello-v2/manifest.yml
    ```

Use the route for each of these applications as the URL of the external services in the Kubernetes manifests.

To clean up afterwards:

  ```sh
  cf delete hello-istio-v1
  cf delete hello-istio-v2
  ```
