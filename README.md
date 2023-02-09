# nginx-controller-api
Configure a load balancer via REST API from NGINX Controller

You can access the NGINX Controller from other hosts via REST API by using the username and the password specified in the install script.

```
# curl -ik --location --request POST 'https://controller.eric.com/api/v1/platform/login' --header 'Content-Type: application/json' --data-raw '{ "credentials": { "type": "BASIC", "username": "<username>", "password": "<password>" }}'
HTTP/1.1 204 No Content
Server: nginx/1.17.6
Date: Wed, 25 Nov 2020 03:28:42 GMT
Connection: keep-alive
Set-Cookie: session=.eJyUjsFO7DAMRf_lrpNRmzeZN2QFC_6AFZsqdVywlDaVkyJViH9HHUbs2dlHPr73E1tlHSQhoIfBdpv8eH5wI3ubIl3s-f91spG8t2Oknly6enYRBsOfjifl-o7QdGODJHXNcR-WODMCYJC4ksrapCw_IBKVbWm3kN5g1fIhiXVo-3oocV6zTDsMJtHa7o-eVQgGOf6Sp63y0hgGPEfJCGAVeqSyNC05s56O_URlvntDLm9ydHCd62zfW-dfOhe6Lvy7vOLrOwAA__8AVF8z.X73Pag.NUycl7yzyW6a1qboHhrKIuV0kyg; Path=/; HttpOnly; Secure; SameSite=Strict
X-Amplify-Id: 6eefe03d264fe1d86323c23b3753d7c6
X-Content-Type-Options: nosniff
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
X-Frame-Options: sameorigin
Content-Security-Policy: default-src 'none'; block-all-mixed-content; script-src 'self' 'unsafe-eval' 'sha256-6ySRbD543mpoTusDX+6lmP/EOV2xDvuLPtxvk39mhAQ='; connect-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline'; font-src 'self'; frame-ancestors 'self'; object-src 'none';
```

You can save the session value and test if you can get the list of running instances from the NGINX Controller.

```
# export session=".eJyUjsFO7DAMRf_lrpNRmzeZN2QFC_6AFZsqdVywlDaVkyJViH9HHUbs2dlHPr73E1tlHSQhoIfBdpv8eH5wI3ubIl3s-f91spG8t2Oknly6enYRBsOfjifl-o7QdGODJHXNcR-WODMCYJC4ksrapCw_IBKVbWm3kN5g1fIhiXVo-3oocV6zTDsMJtHa7o-eVQgGOf6Sp63y0hgGPEfJCGAVeqSyNC05s56O_URlvntDLm9ydHCd62zfW-dfOhe6Lvy7vOLrOwAA__8AVF8z.X73Pag.NUycl7yzyW6a1qboHhrKIuV0kyg"
```

Check if the below instance retrieval is working:
```
# curl -sk --location --request GET 'https://10.200.161.6/api/v1/infrastructure/locations/unspecified/instances' --header 'Content-Type: application/json' --header "Cookie: session=${session}" --data-raw '' | jq 
```

Creating Environment
```
curl -sk --location --request PUT 'https://10.200.161.6/api/v1/services/environments/test-env' --header 'Content-Type: application/json' --header "Cookie: session=${session}" --data-raw '{ "metadata": { "name": "test-env" }}' | jq
```

Creating App
```
 curl -sk --location --request PUT 'https://10.200.161.6/api/v1/services/environments/test-env/apps/test-app' --header 'Content-Type: application/json' --header "Cookie: session=${session}" --data-raw '{ "metadata": { "name": "test-app" } }' | jq
```

Creating Gateway
```
curl -sk --location --request PUT 'https://controller.eric.com/api/v1/services/environments/test-env/gateways/test-gw' --header 'Content-Type: application/json' --header "Cookie: session=${session}" --data-raw '{
    "metadata": {
        "name": "test-gw"
    },
    "desiredState": {
        "ingress": {
            "uris": {
                "http://nginx1.eric.com": {
               }
            },
            "placement": {
               "instanceRefs": [
                    {
                        "ref": "/infrastructure/locations/unspecified/instances/localhost.localdomain""
                    }
                ]
            }
        }
        }
    }
}' | jq
```

Creating Component
```
curl -sk --location --request PUT 'https://controller.eric.com/api/v1/services/environments/test-env/apps/test-app/components/test-comp' --header 'Content-Type: application/json' --header "Cookie: session=${session}" --data-raw '{
    "metadata": {
        "name": "test-comp"
    },
    "desiredState": {
        "ingress": {
            "uris": {
                "/": {
                }
            },
            "gatewayRefs": [
                    {
                        "ref": "/services/environments/test-env/gateways/test-gw"
                    }
                ]
        },
        "backend": {
            "workloadGroups": {
                "test-workloadgroup": {
                    "uris": {
                        "http://10.0.12.2":{
                        },
                        "http://10.0.12.3": {
                        }
                        }
                    }
                }
            }
        }
    }
}' | jq
```
