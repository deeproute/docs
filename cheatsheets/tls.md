# TLS CheatSheet

## Retrieve the expired dates of the TLS Certificate connection

```
echo | openssl s_client -servername clustercontext -connect cluster.my.domain.org:443 2>/dev/null | openssl x509 -noout -dates
```

