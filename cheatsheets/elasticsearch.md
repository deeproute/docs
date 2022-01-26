# ElasticSearch CheatSheet

### See the shards health Status
```
curl http://127.0.0.1:9200/_cluster/health?pretty
curl http://127.0.0.1:9200/_cat/shards?pretty
```

### Check why the shard allocation failed
```
curl http://127.0.0.1:9200/_cluster/allocation/explain?pretty
```

### Retry all that failed
```
curl -XGET "localhost:9200/_cat/indices?v&health=red"
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed'
```

### Check if sync is running
```
curl http://127.0.0.1:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason 2>/dev/null | grep INIT
```

### Retry Failed
```
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed
```