# TLS Certificates

## Generate a CA certificate & Install it in Ubuntu

[Reference](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development)

```sh
mkdir -p ~/certs/ca/company.org
cd ~/certs/ca/company.org

openssl genrsa -des3 -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1825 -out ca.pem

# If ca-certificates package isn't installed: sudo apt install ca-certificates

sudo cp ~/certs/ca/company.org/ca.pem /usr/local/share/ca-certificates/company.org.crt
sudo update-ca-certificates

# Verify if its correctly installed:
awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt | grep company.org

```

## TODO
```
openssl genrsa -out wildcard.k8slab.internal.company.org-2048.key -passout stdin 2048

openssl req -new -sha256 -key wildcard.k8slab.internal.company.org-2048.key -out wildcard.k8slab.
internal.company.org-2048-OV.csr

opessl x509 -in wildcard.k8slab.internal.company.org-2048-OV.crt -text

cat wildcard.k8slab.internal.company.org-2048-OV.crt GlobalSignOrganizationValidationCA-SHA256-G2_R3.pem > wildcard.k8slab.internal.company.org-2048-OV-full-chain.crt

openssl verify -CAfile GlobalSignOrganizationValidationCA-SHA256-G2_R3.pem wildcard.k8slab.internal.company.org-2048-OV.crt

echo | openssl s_client -servername k8slab -connect wildcard.k8slab.internal.company.org:443 2>/dev/null | openssl x509 -noout -dates
```
