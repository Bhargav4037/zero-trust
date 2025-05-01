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
kubectl port-forward svc/keycloak 8080:80
```

## Configure keycloak
Now keycloak will be up and running in the localhost:8080
1. Go to admin consolelogin with username admin and password also admin.
2. After login create a new realm Istio
3. Make some realms roles example admin and user
4. Configure the client with username Istio and click add client
5. Create two users book-admin and book-user assign roles admin and user respectively for these users and create credentials for them.
```
cd ..
kubectl apply -f request_auth.yaml
kubectl apply -f Authorization.yaml
```

## Jwt Authentication and authorisation
1. Generates the jwt access token.
```
curl -X POST -d "client_id=Istio" -d "username=book-user" -d "password=*****" -d "grant_type=password" "http://127.0.0.1:8080/realms/Istio/protocol/openid-connect/token"
```
2. Authorises and authenticates the token
```
curl -X GET -H "host: book-info.test.io" -H "Authorization: Bearer <accessToken>" http://127.0.0.1/getbooks
```
