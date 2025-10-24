# Kubernetes Playground - Minikube

### Esta diseñada para usarse con Minikube, no probe si funciona con Kind o k3s o en algun Cloud 

Este repositorio contiene un playground para practicar Kubernetes, Ingress, TLS, autenticación básica y rewrites.

## Descripción

Se desplegaron **4 subdominios** con sus respectivos backends:

- `web-practica.local` → frontend principal
- `admin.web-practica.local` → admin frontend (con autenticación básica)
- `api.web-practica.local/v1` → API V1 con rewrite
- `api.web-practica.local/v2` → API V2 con rewrite

Cada subdominio tiene su propio Deployment y Service.  
Se usan **Ingress separados** para admin (con auth) y frontend/API (sin auth) para evitar conflictos de TLS y host.  
Todos los subdominios usan **TLS** para HTTPS.


## Cómo usarlo

### 1. Crear TLS secrets

No subir tus claves privadas ni certificados reales a GitHub. Generá tus propios certificados locales:

# Admin TLS
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls/admin.key -out tls/admin.crt -subj "/CN=admin.web-practica.local/O=admin"

# Frontend + API TLS
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls/main.key -out tls/main.crt -subj "/CN=web-practica.local/O=main"

# Crear secretos en Kubernetes
kubectl create secret tls admin-tls --key tls/admin.key --cert tls/admin.crt
kubectl create secret tls main-tls --key tls/main.key --cert tls/main.crt

### 2. Crear secret autenticacion para admin 

htpasswd -c auth/auth adminuser
kubectl create secret generic admin-basic-auth --from-file=auth/auth

### 3. Aplica los deployments y los ingres
kubectl apply -f .....

### 4. Actualizar /etc/hosts apuntando a la IP de Minikube

<minikube-ip> web-practica.local
<minikube-ip> admin.web-practica.local
<minikube-ip> api.web-practica.local

### 5. Probar que funcione 

# Frontend principal
curl -vk https://web-practica.local/

# Admin (pide credenciales)
curl -vk -u adminuser:<tu-password> https://admin.web-practica.local/

# API V1
curl -vk https://api.web-practica.local/v1/test

# API V2
curl -vk https://api.web-practica.local/v2/test


## Notas importantes

Rewrites en la API permiten mantener compatibilidad con rutas internas.

Autenticación básica solo aplica al subdominio admin.

TLS asegura conexiones HTTPS en todos los subdominios.

Diseñado para Minikube local, no requiere cluster remoto.

Archivos sensibles (auth y TLS) se ignoran en GitHub mediante .gitignore.






