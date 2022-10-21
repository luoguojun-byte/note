1. curl -k -X 'POST' -v https://nova-api.trystack.org:5443/v2.0/tokens -d '{"auth":{"passwordCredentials":{"username": "joecool", "password":"coolword"}, "tenantId":"5"}}' -H 'Content-type: application/json'

```
curl -k -X 'POST' -v http://10.0.0.70:5000/v3/tokens -d '{"auth":{"passwordCredentials":{"username": "admin", "password":"000000"}, "tenantId":"687ce0c4f33146949292c2df3c22f61a"}}' -H 'Content-type: application/json'
```



```
curl -i 'http://10.0.0.100:5000/v2.0/tokens' -X POST -H "Content-Type: application/json" -H "Accept: application/json"  -d '{"auth": {"tenantName": "admin", "passwordCredentials": {"username": "admin", "password": "000000"}}}'
```



```

方法1：

#发起一个post请求, json格式 
#会获取token以及所有api的调用地址

curl -i -X POST -H "Content-type: application/json" 
-d '{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "default"
                    },
                    "name": "admin",
                    "password": "123456"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "name": "default"
                },
                "name": "admin"
            }
        }
    }
}' http://10.0.0.11:5000/v3/auth/tokens




方法2：

token=`openstack token issue|awk 'NR==5{print $4}’`

echo $token


```

