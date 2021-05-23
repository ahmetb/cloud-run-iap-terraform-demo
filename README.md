# IAP Terraform Sample for Cloud Run

This sample deploys a [Cloud Run](https://cloud.run/) service with VPC
[ingress controls] that only allows traffic from Cloud HTTPS load balancer that
has [IAP (Identity Aware Proxy)][iap] enabled.

[iap]: https://cloud.google.com/iap
[ingress controls]: https://cloud.google.com/run/docs/securing/ingress

IAP authenticates users with a Google account (or other external IdP) and
checks if the user is allowed to access the deployed service.

## Prerequisites
You must have a domain name registered, for which you can modify certain DNS settings. You can use a subdomain as well.

## Deploy

### Create a project
Create a new project, if you don't want to re-use an existing project. For example:
```sh
gcloud projects create cloud-run-iap-terraform-demo-1 --name="IAP Cloud Run Demo" --set-as-default
```

If this project isn't set as your current default project, use:
```sh
gcloud config set project cloud-run-iap-terraform-demo-1
```

### Enable billing
Enable billing, if you haven't done this yet, for the project. First get your billing account ID:
```sh
gcloud beta billing accounts list
```

This returns something like:
```sh
ACCOUNT_ID            NAME                OPEN  MASTER_ACCOUNT_ID
0X0X0X-0X0X0X-0X0X0X  My Billing Account  True
```

Now enable billing for your new project:
```sh
gcloud beta billing projects link cloud-run-iap-terraform-demo-1 --billing-account 0X0X0X-0X0X0X-0X0X0X
```

### Enable APIs
Enable the following APIs in your (currently selected) project:

```sh
gcloud services enable \
    run.googleapis.com \
    containerregistry.googleapis.com \
    compute.googleapis.com \
    iap.googleapis.com \
    vpcaccess.googleapis.com
```

It might take a minute to complete. Optionally verify that these services are enabled for your current project:
```sh
gcloud services list
```

### Create an OAuth2 web application
Create OAuth2 web application (and note its client_id and client_secret)
from https://console.cloud.google.com/apis/credentials.

If it asks you to Configure a consent screen first, then do so:
- Choose 'External' for *User Type*
- Fill in 'App name', e.g. `IAP Cloud Run Demo`
- Select your email address from the dropdown, for 'User support email'
- You can leave the 'App domain' section blank  
- Fill in your email address at the 'Developer contact information'
- Click on 'save and continue' on the next screens and then the 'Credentials' menu link in the left menu bar

### Execute Terraform
First initialize the Terraform:

```sh
terraform init
```

(Optional) preview what resources will be created:
```sh
terraform apply -var project_id=<project-id> \
    -var region=<region> \
    -var domain=<domain> \
    -var lb_name=<lb_name> \
    -var iap_client_id=<client_id> \
    -var iap_client_secret=<client_secret>
```

Deploy using Terraform, this will take several minutes to complete:

```sh
terraform apply -var project_id=<project-id> \
    -var region=<region> \
    -var domain=<domain> \
    -var lb_name=<lb_name> \
    -var iap_client_id=<client_id> \
    -var iap_client_secret=<client_secret> \
    -auto-approve
```

In this command fill in:
- project-id: the project id, e.g. `cloud-run-iap-terraform-demo-1`
- region: the region, this is optional, will default to `us-central1`
- domain: the domain name you want to use, e.g. `corpapp.ahmet.dev`
- lb_name: name for the load balancer, this is optional, will default to `iap-lb`
- iap_client_id: the client id for the oAuth client you created earlier
- iap_client_secret: the client secret for the oAuth client you created earlier. You can view it by editing the oAuth client.

#### Troubleshooting errors
Note: if you get an error saying `Output refers to sensitive values`, add `sensitive = true` to `backend_services`
in `.terraform/modules/lb-http/modules/serverless_negs/outputs.tf`.

If you get an error like `Error: Error creating Connector: googleapi: Error 403: The caller does not have permission`,
it means the account which is used to execute the terraform, does not have enough permissions. You can fix this by running:
```sh
gcloud auth application-default login
```

If that still doesn't solve the issue, you can view the service account of your project in the GCP console under
`IAM & Admin` -> 'Service Accounts' named e.g. `Default compute service account`. From there you can troubleshoot which
permissions might be missing.

### Post-deployment steps
After the deployment is complete:

1. Configure DNS records of the used domain name with the given IP address. In other words, add a record named `www` or
   your subdomain name, of type `A` pointing to your ipv4 IP address, which was the output of your terraform command.
   **It will take 10-30 minutes until the load balancer starts functioning.**
1. Go back to Credentials page and add a the `oauth2_redirect_uri` output to the
   Authorized Redirect URIs of the web application you created here.

Now, you should be able to authenticate to your web application based on
the users/group specified in [`main.tf`](./main.tf) by visiting your domain.

## Cleanup

:warning: Do not lose the `.tfstate` file created in this directory.

Run the previous `terraform apply` command as `terraform destroy` to clean up
the created resources.

------

This is not an official tutorial and can go out of date. Please read the
documentation for more up-to-date info.
