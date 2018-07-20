## Getting Started

The following will describe how to install and configure the DigitalOcean blockstore plugin for Ark and provide a usage example.

### Prerequisites

* [Kubernetes cluster](https://stackpoint.io/clusters/new?provider=do)
* DigitalOcean account and resources
  * [API personal access token](https://www.digitalocean.com/docs/api/create-personal-access-token/)
  * [Spaces access keys](https://www.digitalocean.com/docs/spaces/how-to/administrative-access/)
  * Spaces bucket
  * Spaces bucket region
* [Heptio Ark](https://heptio.github.io/ark/master/quickstart.html) prerequisites

### Install the plugin

1. Complete the Heptio Ark prerequisites mentioned above. This generally involves apply the `00-prereqs.yaml` available in the Ark repository:

    ```
    kubectl apply -f 00-prereqs.yaml
    ```

2. Update the `examples/credentials-ark` with your Spaces access and secret keys. The file will look like the following:

    ```
    [default]
    aws_access_key_id=<AWS_ACCESS_KEY_ID>
    aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
    ```

3. Create a Kubernetes `cloud-credentials` secret containing the `credentials-ark` and DigitalOcean API token.

    ```
    kubectl create secret generic cloud-credentials \
        --namespace heptio-ark \
        --from-file cloud=credentials-ark \
        --from-literal digitalocean_token=<DIGITALOCEAN_TOKEN>
    ```

4. Update the `example/10-ark-config.yaml` with the Spaces API URL, bucket, and region and apply the configuration:

    ```
    kubectl apply -f 10-ark-config.yaml
    ```

5. Now apply the Ark deployment.

    ```
    kubectl apply -f 20-deployment.yaml
    ```

6. Finally add the `ark-blockstore-digitalocean` plugin to Ark.

    ```
    ark plugin add quay.io/stackpoint/ark-blockstore-digitalocean:latest
    ```

### Backup and restore example

1. Apply the Nginx `example/nginx-pv.yml` config that uses persistent storage for the log path.

    ```
    kubectl apply -f nginx-pv.yml
    ```

2. Once Nginx deployment is running and available, create a backup using Ark.

    ```
    ark backup create nginx-backup --selector app=nginx
    ark backup describe nginx-backup
    ```

3. The config files should appear in the Spaces bucket and a snapshot taken of the persistent volume. Now you can simulate a disaster by deleting the `nginx-example` namespace.

    ```
    kubectl delete namespace nginx-example
    ```

4. The `nginx-data` backup can now be restored.

    ```
    ark restore create --from-backup nginx-data
    ```

### Build container

make container IMAGE=quay.io/stackpoint/ark-blockstore-digitalocean
