### 第一次查看token值

```
curl -i -X POST -H "Content-type: application/json" -d '{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "id": "151fa95eb4d14ad59905887399a34675",
                    "password": "000000"
                }
            }
        },
        "scope": "unscoped"
    }
}' http://10.0.0.70:5000/v3/auth/tokens
```

### 实战

```
import requests
import json
myurl="http://10.0.0.70"
body={
    "auth":{
        "identity":{
            "methods":["password"],
            "password":{
                "user":{
                    "id":"151fa95eb4d14ad59905887399a34675",
                    "password":"000000"
                }
            }
        },
        "scope":{
            "project":{
                "id":"687ce0c4f33146949292c2df3c22f61a"
            }
        }
    }
}
headers={}
def get_token():
    url =myurl+":5000/v3/auth/tokens"
    re = requests.post(url,headers=headers,data=json.dumps(body)).headers["X-subject-Token"]
    print(re)
    return re
def flavor_create():
    url =myurl+":8774/v2.1/flavors"
    headers["X-Auth-Token"]=get_token()
    body={
        "flavor":{
            "name":"centos7",
            "id":111,
            "vcpus":1,
            "ram":2048,
            "disk":40,
        }
    }
    re=requests.post(url,data=json.dumps(body),headers=headers).json()
    print(re)
    return re
def flavor_delete():
    url=myurl+":8774/v2.1/flavors/302"
    headers["X-Auth-Token"]=get_token()
    re=requests.delete(url,headers=headers).json()
    print(re)
    return re
def image_create():
    url=myurl+":9292/v2/images"
    body={
        "container_format": "bare",
        "disk_format": "qcow2",
        "name": "test1",
    }
    headers["X-Auth-Token"] = get_token()
    re=requests.post(url,data=json.dumps(body),headers=headers).json()
    # re = requests.get(url,headers=headers).json()
    print(re)
    return re
image_create()
get_token()
```

