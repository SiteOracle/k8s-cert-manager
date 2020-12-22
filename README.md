# k8s-cert-manager
DNS &amp; TLS configuration files for Site Oracle on k8s

### TODO: this repo demonstartes using static credentials, but it would be preferable to use GKE Workload Identity

[Source](https://cert-manager.io/docs/configuration/acme/dns01/google/)

## Create service account
```bash
$ PROJECT_ID=site-oracle-gcp-account  # fake
$ gcloud iam service-accounts create dns01-solver --display-name "dns01-solver"
```

# Give service-account DNS admin permissions in GCP CloudDNS
```bash
$ gcloud projects add-iam-policy-binding $PROJECT_ID \
   --member serviceAccount:dns01-solver@$PROJECT_ID.iam.gserviceaccount.com \
   --role roles/dns.admin
```

## Create a Service Account Secret (static auth method)
```bash
$ gcloud iam service-accounts create dns01-solver
```

## Save service-account key.json as k8s secret
```bash
$ gcloud iam service-accounts keys create key.json \
   --iam-account dns01-solver@$PROJECT_ID.iam.gserviceaccount.com
$ kubectl create secret generic clouddns-dns01-solver-svc-acct \
   --from-file=key.json
```

## Deploy cert-manager as a Cluster Issuer with letsencrypt as the ACME server 
```bash
$ kubectl apply -f cluster_issuer.yml
```

## Deploy ingress with TLS DNS records (note annotation under metadata)
```bash
$ kubectl apply -f example_ing.yml
```

