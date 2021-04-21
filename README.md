# IAP Terraform Sample for Cloud Run

This sample deploys a [Cloud Run](https://cloud.run/) service with VPC
[ingress controls] that only allows traffic from Cloud HTTPS load balancer that
has [IAP (Identity Aware Proxy)][iap] enabled.

[iap]: https://cloud.google.com/iap
[ingress controls]: https://cloud.google.com/run/docs/securing/ingress

IAP authenticates users with a Google account (or other external IdP) and
checks if the user is allowed to access the deployed service.

## Deploy

Enable the following APIs in your project:

```sh
gcloud services enable \
    run.googleapis.com \
    containerregistry.googleapis.com \
    compute.googleapis.com \
    iap.googleapis.com \
    vpcaccess.googleapis.com
```

Create OAuth2 web application (and note its client_id and client_secret)
from https://console.cloud.google.com/apis/credentials.

Deploy using Terraform:

```sh
terraform init
```

```sh
terraform apply -var project_id=ahmet-temp \
    -var domain=<domain> \
    -var lb_name=corp-app \
    -var iap_client_id=<client_id> \
    -var iap_client_secret=<client_secret> \
    -auto-approve
```

After the deployment is complete:

1. Configure DNS records of the used domain name with the given IP address.
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
