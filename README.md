# kubernetes-loadbalancer-tests

## Create the objects pod and service LB files
```
$ cat pod-nginx-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    app: www-api
spec:
  containers:
  - name: www
    image: nginx:1.20.1
    ports:
    - containerPort: 80

$ cat pod-nginx-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    app: www-api
spec:
  containers:
  - name: www
    image: nginx:1.20.1
    ports:
    - containerPort: 80

$ cat service-loadbalancer-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxloadbalancer
spec:
  selector:
    app: www-api
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31200

```

# Create the pods and create a file /usr/share/nginx/html/hostname.html to use for curl
```
kubectl -n dev apply -f pod-nginx.yaml 
kubectl -n dev apply -f pod-nginx-2.yaml  

kubectl -n dev exec nginx --  /bin/bash -c  '/bin/echo "nginx" >  /usr/share/nginx/html/hostname.html'
kubectl -n dev exec nginx --  /bin/cat /usr/share/nginx/html/hostname.html
nginx

kubectl -n dev exec nginx-2 --  /bin/bash -c  '/bin/echo "nginx-2" >  /usr/share/nginx/html/hostname.html'
kubectl -n dev exec nginx-2 --  /bin/cat /usr/share/nginx/html/hostname.html
```

## Create the LB service
```
kubectl -n dev apply -f service-loadbalancer-nginx.yaml
```

## Get informations for the Cluster entry en service LB port
```
kubectl get no -o wide|grep multinode-demo|awk '{print $6}'|head -n 1
192.168.49.2

kubectl -n dev get svc |grep nginxloadbalancer|awk '{print $5}'|awk -F':'  '{print $2}'|awk -F'/'  '{print $1}'
31200

curl http://192.168.49.2:31200/hostname.html
```

## Lanch the test on the LB
```
while true; do curl http://192.168.49.2:31200/hostname.html ; sleep 1; done
nginx
nginx
nginx-2
nginx-2
nginx
nginx
nginx
nginx
nginx-2
nginx-2
nginx-2
nginx
nginx
nginx-2
```
