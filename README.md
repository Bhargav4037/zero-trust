# zero-trust-architure
Make sure you have istio, helm and minikube installed in your system.

## Configure and install requirements.
1. Once you have these start the minikube by this command.
   ```
   minikube start
   ```
2. Install the Istio on your Kubernetes cluster using istioctl with a demo configuration profile
   ```
   istioctl install --set profile=demo -y
   ``` 
For the testing sake we will use this simple book-info app which requires mysql db to store information about books and expose two api endpoints to add and view books.

## Installing book-info app on kubernetes
```
git clone <Link-to-this-repo>
cd istio-keycloak
kubectl apply -f app/database.yaml
kubectl apply -f app/app.yaml
```
Now, we will set up an Istio gateway and virtual service to access the app. Gateway allows us to configure ingress traffic to our application from external systems and users. Plus, the Istio gateway does not include any traffic routing configuration so we have to create a virtual service to route traffic coming in from the Istio gateway to the backend kubernetes service.
```
kubectl apply -f istio-manifests/ingressGateway.yaml
kubectl apply -f istio-manifests/virtualService.yaml
```

## Access API endpoints to add and view books

First port forward the book-info app service to access the endpoints or start the minikube tunnel in seperate window to act as a load balancer.

```
minikube tunnel
```

1. Add Book
   ```
   curl -X POST http://localhost/addbook -d '{"isbn": 9781982156909, "title": "The Comedy of Errors", "synopsis": "The authoritative edition of The Comedy of Errors from The Folger Shakespeare Library, the trusted and widely used Shakespeare series for students and general readers", "authorname": "William Shakespeare", "price": 10.39}'
   ```
2. View Books
   ```
   curl -X GET http://localhost/getbooks
   ```

## Setup Keycloak for JWT authentication

Now that we have helm we can set up keycloak.
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
cd keycloak-authentication-tutorial-helm-kubernetes
helm install keycloak bitnami/keycloak -f helm/values.yaml
```
