


This document shows how we can setup the certificate for the analytics.rayprod.monitoring.ray.life	domain. 

The certificate is generated using certbot. We will renew the certificate using dns validation and then place it in a bitbucket repository (onprem branch ) from where flux will sync the file.

setup the credentials in the CLI.

```shell
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."
```

You can copy the AWS credentials from the SSO start page. 

when you run `aws sts get-caller-identity`, it should give a valid response like

```json
{
    "UserId": "AROAYPGMAAYGKRXOBW5YL:sagar@cloudsolitaire.com",
    "Account": "582395889164",
    "Arn": "arn:aws:sts::582395889164:assumed-role/AWSReservedSSO_AdministratorAccess_eb2f27b39d0a539a/sagar@cloudsolitaire.com"
}
```

Make sure you have python and pip on your machine.

```shell
pip install certbot
pip install certbot-dns-route53
```


Use this certbot command to generate the certificate

`certbot certonly --dns-route53-propagation-seconds 30 --dns-route53 -d analytics.rayprod.monitoring.ray.life --non-interactive --agree-tos -m chirayu.patel@ray.life --config-dir ./raydir/config --work-dir ./raydir/workdir --logs-dir ./raydir/logsdir


```shell
Successfully received certificate.
Certificate is saved at: /home/sagargulabani/dev/scratchpad/raydir/config/live/rayprod.analytics.monitoring.ray.life/fullchain.pem
Key is saved at:         /home/sagargulabani/dev/scratchpad/raydir/config/live/rayprod.analytics.monitoring.ray.life/privkey.pem
This certificate expires on 2024-02-25.
These files will be updated when the certificate renews.
```

Once certificate is generated put this file at this location, generated certificate can be base64 encoded.
The following files need to be taken 
1. fullchain.pem
2. privkey.pem

to base64 encode the two files (one by one), use this command

`cat file.pem | base64`


Then push the contents to this location.

https://bitbucket.org/raywifi/k8s/src/onprem/workloads/addons/base/rayprod-prometheus-tenant/rayprod-analytics-tls-cert.yaml

Once done ensure that the cert is synced.

Login as root user into the machine from where you can use the `kubectl` CLI.

Ensure that there is no error for this output and the git commit matches the commit in bitbucket.

`kubectl get kustomizations -n flux-system`

`kubectl get secret rayprod-analytics-tls-cert -o yaml -n default`
