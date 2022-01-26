# TLS Certificates

```
openssl genrsa -out wildcard.k8slab.internal.company.org-2048.key -passout stdin 2048

openssl req -new -sha256 -key wildcard.k8slab.internal.company.org-2048.key -out wildcard.k8slab.
internal.company.org-2048-OV.csr

opessl x509 -in wildcard.k8slab.internal.company.org-2048-OV.crt -text

cat wildcard.k8slab.internal.company.org-2048-OV.crt GlobalSignOrganizationValidationCA-SHA256-G2_R3.pem > wildcard.k8slab.internal.company.org-2048-OV-full-chain.crt

openssl verify -CAfile GlobalSignOrganizationValidationCA-SHA256-G2_R3.pem wildcard.k8slab.internal.company.org-2048-OV.crt

echo | openssl s_client -servername k8slab -connect wildcard.k8slab.internal.company.org:443 2>/dev/null | openssl x509 -noout -dates
```
