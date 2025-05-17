# This application deploy mongodb and mongo express using k8s and helm

# Add TLS AND SSL certificate to kubernestes for mongodb and mongoexpress

## Steps to follow:
    - Generate the CA Key and Certificate: openssl genrsa -out ca.key 4096
        openssl req -x509 -new -nodes -key ca.key -sha256 -days 365 -out ca.crt -subj "/CN=MyMongoCA"

    - Generate the MongoDB Server Key and CSR:
             openssl genrsa -out mongo.key 2048
             openssl req -new -key mongo.key -out mongo.csr -subj "/CN=your.mongo.service.cluster.local"

    - Sign the CSR with your CA:
             openssl x509 -req -in mongo.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mongo.crt -days 365 -sha256

    - Create a Combined PEM File:
             cat mongo.key mongo.crt > mongo.pem

    - You can now create the Kubernetes secret:
            kubectl create secret generic mongo-ssl --from-file=mongo.pem=./mongo.pem --from-file=ca.crt=./ca.crt

    - Base64 Encode the Files:
            - base64 -w 0 mongo.pem > mongo.pem.b64
            - base64 -w 0 ca.crt > ca.crt.b64

# reate mongo-ssl-secret.yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: mongo-ssl
    type: Opaque
    data:
    mongo.pem: <base64-encoded-content-of-mongo.pem>
    ca.crt: <base64-encoded-content-of-ca.crt>

# Apply manuel or use helm upgrade <repo-name>
    kubectl apply -f mongo-ssl-secret.yaml

# Install iamserviceAccount using helm & install eks aws ALB Controller.