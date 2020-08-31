# Etcd

```shell
1.导入etcd api 版本
2.key需要base64
3.https://doczhcn.gitbook.io/etcd/index/index/api_grpc_gateway

curl -i --cacert /opt/kubernetes/ssl/ca.crt --cert /opt/kubernetes/ssl/server.crt --key /opt/kubernetes/ssl/server.key  https://shcbingcentos77vm01.hpeswlab.net:4001/v3alpha/kv/put -X POST -d '{"key":"YmluZwo=","value":"MTIzNDU2Cg=="}'


etcdctl --endpoints https://shcbingcentos77vm01.hpeswlab.net:4001 --cacert=/opt/kubernetes/ssl/ca.crt --key=/opt/kubernetes/ssl/server.key  --cert=/opt/kubernetes/ssl/server.crt get --prefix bing 
```

