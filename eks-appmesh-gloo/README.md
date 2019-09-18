

```bash
#create upstreamgroup for color blue
glooctl create upstreamgroup  \
        --name upstreamgroup-blue \
        -n appmesh-demo \
        --weighted-upstreams 'gloo-system.appmesh-demo-colorteller-blue-9080=100'

#creat hpa for blue
kubectl apply -f hpa.yaml

#deploy the load testing service to generate traffic during the canary analysis:
helm upgrade -i flagger-loadtester flagger/loadtester \
            --namespace=appmesh-demo

#check HPA all good so far
kubectl -n appmesh-demo get horizontalpodautoscaler.autoscaling/hpa-blue


#recreate blue route
glooctl add route --path-prefix /appmesh/blue \
      --prefix-rewrite / \
      --upstream-group-name upstreamgroup-blue \
      --upstream-group-namespace appmesh-demo

#deploy bluesteel
 kubectl apply -f color-bluesteel.yaml

# create bluesteel route with header
glooctl add route --path-prefix /appmesh/blue \
      --prefix-rewrite / \
      --upstream-group-name upstreamgroup-blue \
      --upstream-group-namespace appmesh-demo
 
 
glooctl add route --path-prefix /appmesh/blue \
      --prefix-rewrite / \
      --upstream-group-name upstreamgroup-blue \
      --upstream-group-namespace appmesh-demo \
      --header x-canary=true
 #test curl blue
```