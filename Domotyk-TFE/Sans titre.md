## clef privée ca
sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out ca.key

## certif auto-signé
sudo openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=MyMosquittoCA"

## fichier san créé

[ req ]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[ req_distinguished_name ]
CN = 192.168.3.100

[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 192.168.3.100

## clé privée server

sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
sudo openssl req -new -key server.key -out server.csr -config san.cnf

## certif signature

sudo openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key  -CAcreateserial -out server.crt -days 365  -extensions v3_req -extfile san.cnf

## Clients 

sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-mqtt-explorer.key

sudo openssl req -new -key client-mqtt-explorer.key -out client-mqtt-explorer.csr -subj "/CN=mqtt-explorer"

sudo openssl x509 -req -in client-mqtt-explorer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-mqtt-explorer.crt -days 3650

sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-reader.key
sudo openssl req -new -key client-esp-reader.key -out client-esp-reader.csr -subj "/CN=esp32-reader"
sudo openssl x509 -req -in client-esp-reader.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-reader.crt -days 3650

sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out backend.key
sudo openssl req -new -key backend.key -out backend.csr -subj "/CN=backend"
sudo openssl x509 -req -in backend.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out backend.crt -days 3650

sudo openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out client-esp-recorder.key
sudo openssl req -new -key client-esp-recorder.key -out client-esp-recorder.csr -subj "/CN=esp32-recorder"
sudo openssl x509 -req -in client-esp-recorder.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client-esp-recorder.crt -days 3650

