# Ambassador Cheatsheet

## Host management

### Get all hosts without a request policy

```
k get host -A -ojson | jq -r '.items[] | select(.spec.requestPolicy == null) | { namespace: .metadata.namespace, host: .spec.hostname }'

OR just the hostname

k get host -A -ojson | jq -r '.items[] | select(.spec.requestPolicy == null) | [ .spec.hostname } | @tsv'

```
