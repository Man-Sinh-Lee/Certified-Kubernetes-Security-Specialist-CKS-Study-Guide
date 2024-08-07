kubectl apply -f nginx.yaml

kubectl get all -n ingress

kubectl run tm --image radial/busyboxplus:curl --restart=Never -it --rm curl accounting-service.ingress.svc.cluster.local

openssl req -nodes -new -x509 -keyout accounting.key -out accounting.crt -subj "/CN=accounting.tls"

kubectl create secret tls accounting-secret --cert=accounting.crt --key=accounting.key -n ingress

kubectl create ingress accounting-ingress \
--rule="accounting.905418456792.realhandsonlabs.net/*=accounting-service:80, \
tls=accounting-secret" -n ingress