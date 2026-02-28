**Online Boutique** is a cloud-first microservices demo application.  The application is a
web-based e-commerce app where users can browse items, add them to the cart, and purchase them.

As a test-task we have modified this demo to expand its DevOps approaches.

Google uses this application to demonstrate how developers can modernize enterprise applications using Google Cloud products, including: [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine), [Cloud Service Mesh (CSM)](https://cloud.google.com/service-mesh), [gRPC](https://grpc.io/), [Cloud Operations](https://cloud.google.com/products/operations), [Spanner](https://cloud.google.com/spanner), [Memorystore](https://cloud.google.com/memorystore), [AlloyDB](https://cloud.google.com/alloydb), and [Gemini](https://ai.google.dev/). This application works on any Kubernetes cluster.

## Architecture

**Online Boutique** is composed of 11 microservices written in different languages that talk to each other over gRPC.

### Cloud Architecture

The application is deployed on **Google Cloud Platform** using the following stack:

- **Infrastructure (IaC):** All cloud resources are provisioned with Terraform — GKE cluster, namespaces, ingress, TLS certificates, and the monitoring stack. A single `terraform apply` brings up the full environment; `terraform destroy` tears it down cleanly.

- **Kubernetes (GKE):** A single GKE cluster (zonal, `us-central1-b`) hosts two isolated environments in separate namespaces:
  - `staging` — includes a load generator for traffic simulation
  - `prod` — network policies enabled, no load generator

- **Ingress & TLS:** Nginx Ingress Controller routes external traffic. TLS certificates are managed automatically via cert-manager with Let's Encrypt (`staging.koma-net.com`, `prod.koma-net.com`).

- **CI/CD (GitHub Actions):** Automated pipelines for `paymentservice` and `emailservice` — on every push to `main` the workflow builds a Docker image, pushes it to Google Artifact Registry, deploys to `staging`, waits for rollout, then deploys to `prod`.

- **Monitoring:** Prometheus + Grafana deployed via Helm (`kube-prometheus-stack`). Four custom dashboards cover reliability and scalability metrics: pod health, CPU/memory usage vs limits, throttling, restart history, and network throughput.

[![High-level Cloud Architecture](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./protos` directory](/protos).
Find **Microservices** at the [`./src` directory](/src).

## Screenshots
### Cloud Deployment
|1|2|
| :---------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: |
| [![Screenshot of deployment 1](/docs/img/deployment-1.png)](/docs/img/deployment-1.png) | [![Screenshot of deployment 2](/docs/img/deployment-2.png)](/docs/img/deployment-2.png) |

### Activated CI/CD
|1|2|
| :-----------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: |
| [![Screenshot of CI/CD 1](/docs/img/cicd-1.png)](/docs/img/cicd-1.png) | [![Screenshot of CI/CD 2](/docs/img/cicd-2.png)](/docs/img/cicd-2.png) |

### Monitoring and Dashboards
|1|2|3|
| :-----------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------: |
| [![Screenshot of dashboard 1](/docs/img/dashboard-1.png)](/docs/img/dashboard-1.png) | [![Screenshot of dashboard 2](/docs/img/dashboard-2.png)](/docs/img/dashboard-2.png) | [![Screenshot of dashboard 3](/docs/img/dashboard-3.png)](/docs/img/dashboard-3.png) | 

## Showcase
[![Showcase video](/docs/img/showcase-thumbnail.png)](/docs/mov/showcase.mp4)

## Use Terraform to deploy Online Boutique on a GKE cluster

This page walks you through the steps required to deploy the [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) sample application on a [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) cluster using Terraform.

### Prerequisites

1. [Create a new project or use an existing project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#console) on Google Cloud, and ensure [billing is enabled](https://cloud.google.com/billing/docs/how-to/verify-billing-enabled) on the project.

### Deploy the sample application

1. Clone the Github repository.

    ```bash
    git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
    ```

1. Move into the `terraform/` directory which contains the Terraform installation scripts.

    ```bash
    cd microservices-demo/terraform
    ```

1. Open the `terraform.tfvars` file and replace `<project_id_here>` with the [GCP Project ID](https://cloud.google.com/resource-manager/docs/creating-managing-projects?hl=en#identifying_projects) for the `gcp_project_id` variable.

1. (Optional) If you want to provision a [Google Cloud Memorystore (Redis)](https://cloud.google.com/memorystore) instance, you can change the value of `memorystore = false` to `memorystore = true` in this `terraform.tfvars` file.

1. Initialize Terraform.

    ```bash
    terraform init
    ```

1. See what resources will be created.

    ```bash
    terraform plan
    ```

1. Create the resources and deploy the sample.

    ```bash
    terraform apply
    ```

    1. If there is a confirmation prompt, type `yes` and hit Enter/Return.

    Note: This step can take about 10 minutes. Do not interrupt the process.

Once the Terraform script has finished, you can locate the frontend's external IP address to access the sample application.

- Option 1:

    ```bash
    kubectl get service frontend-external | awk '{print $4}'
    ```

- Option 2: On Google Cloud Console, navigate to "Kubernetes Engine" and then "Services & Ingress" to locate the Endpoint associated with "frontend-external".

### Clean up

To avoid incurring charges to your Google Cloud account for the resources used in this sample application, either delete the project that contains the resources, or keep the project and delete the individual resources.

To remove the individual resources created for by Terraform without deleting the project:

1. Navigate to the `terraform/` directory.

1. Set `deletion_protection` to `false` for the `google_container_cluster` resource (GKE cluster).

   ```bash
   # Uncomment the line: "deletion_protection = false"
   sed -i "s/# deletion_protection/deletion_protection/g" main.tf

   # Re-apply the Terraform to update the state
   terraform apply
   ```

1. Run the following command:

   ```bash
   terraform destroy
   ```

   1. If there is a confirmation prompt, type `yes` and hit Enter/Return.
   
## Instructions to Deploy Manually (GKE)

1. Ensure you have the following requirements:
   - [Google Cloud project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project).
   - Shell environment with `gcloud`, `git`, and `kubectl`.

2. Clone the latest major version.

   ```sh
   git clone --depth 1 --branch v0 https://github.com/GoogleCloudPlatform/microservices-demo.git
   cd microservices-demo/
   ```

   The `--depth 1` argument skips downloading git history.

3. Set the Google Cloud project and region and ensure the Google Kubernetes Engine API is enabled.

   ```sh
   export PROJECT_ID=<PROJECT_ID>
   export REGION=us-central1
   gcloud services enable container.googleapis.com \
     --project=${PROJECT_ID}
   ```

   Substitute `<PROJECT_ID>` with the ID of your Google Cloud project.

4. Create a GKE cluster and get the credentials for it.

   ```sh
   gcloud container clusters create-auto online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Creating the cluster may take a few minutes.

5. Deploy Online Boutique to the cluster.

   ```sh
   kubectl apply -f ./release/kubernetes-manifests.yaml
   ```

6. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
   cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
   checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
   currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
   emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
   frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
   loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
   paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
   productcatalogservice-557d474574-888kr   1/1     Running   0          3m
   recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
   redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
   shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
   ```

7. Access the web frontend in a browser using the frontend's external IP.

   ```sh
   kubectl get service frontend-external | awk '{print $4}'
   ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Online Boutique.

8. Congrats! You've deployed the default Online Boutique. To deploy a different variation of Online Boutique (e.g., with Google Cloud Operations tracing, Istio, etc.), see [Deploy Online Boutique variations with Kustomize](#deploy-online-boutique-variations-with-kustomize).

9. Once you are done with it, delete the GKE cluster.

   ```sh
   gcloud container clusters delete online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Deleting the cluster may take a few minutes.

## Additional deployment options

- **Istio / Cloud Service Mesh**: [See these instructions](/kustomize/components/service-mesh-istio/README.md) to deploy Online Boutique alongside an Istio-backed service mesh.
- **Non-GKE clusters (Minikube, Kind, etc)**: See the [Development guide](/docs/development-guide.md) to learn how you can deploy Online Boutique on non-GKE clusters.
- **AI assistant using Gemini**: [See these instructions](/kustomize/components/shopping-assistant/README.md) to deploy a Gemini-powered AI assistant that suggests products to purchase based on an image.
- **And more**: The [`/kustomize` directory](/kustomize) contains instructions for customizing the deployment of Online Boutique with other variations.

## Documentation

- [Development](/docs/development-guide.md) to learn how to run and develop this app locally.
