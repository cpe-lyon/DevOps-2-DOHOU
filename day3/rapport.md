## ELK

### Scale Up : Que constate-t-on ?

```
epoch      timestamp cluster             status node.total node.data shards pri relo init unassign unassign.pri pending_tasks max_task_wait_time active_shards_percent
1733300998 08:29:58  medhy-dohou-elastic green           3         3     66  33    2    0        0            0             0                  -                100.0%
```

Le node.total est passé à 3

```
shards shards.undesired write_load.forecast disk.indices.forecast disk.indices disk.used disk.avail disk.total disk.percent host         ip           node                             node.role
22                0                 0.0                 2.8mb        2.8mb      21mb    952.4mb    973.4mb            2 10.60.46.227 10.60.46.227 medhy-dohou-elastic-es-default-1 cdfhilmrstw
22                0                 0.0               863.1kb      863.1kb    18.8mb    954.6mb    973.4mb            1 10.60.11.159 10.60.11.159 medhy-dohou-elastic-es-default-0 cdfhilmrstw
22                0                 0.0                 3.1mb        3.1mb    21.1mb    952.2mb    973.4mb            2 10.60.30.229 10.60.30.229 medhy-dohou-elastic-es-default-2 cdfhilmrstw
```

3 noeuds apparaissent dans l'allocation.