# All About openssl
## Cheatsheet for OpenSSL
### Step 1 : Download openssl 
### Step 2 : validate installation :
	openssl version -a

### generate private key
	openssl genrsa -out ssl_pvt.key 2048
### generate public key
	openssl rsa -in ssl_pvt.key -putout -out ssl_pub.key

### generate csr 
	openssl req -new -key ssl_pvt.key -out ssl.csr
	[common name = must match with hostname or ip]
	password can be put blank , default password : changeme
	
### verify 
	openssl req -text  -ssl.csr -noout -verify

### self signining
	openssl x509 -in ssl.csr -out ssl.crt -req -signkey ssl_pvt.key -days 3650


# Alternate approach
### Generate CA
	openssl genrsa -out cert_key.pem 4096
	openssl req -new -x509 -sha256 -days 6300 -key ca-key.pem -out ca.pem
#### Optional Stage: View Certificate's Content
	openssl x509 -in ca.pem -text
	openssl x509 -in ca.pem -purpose -noout -text

### Install the CA Cert as a trusted root CA
#### Assuming the path to your generated CA certificate as `C:\ca.pem`, run: powershell
	Import-Certificate -FilePath "C:\ca.pem" -CertStoreLocation Cert:\LocalMachine\Root
#### OR In Command Prompt, run:
	certutil.exe -addstore root C:\ca.pem

### Generate Certificate
#### Create a Certificate Signing Request (CSR)
	openssl req -new -sha256 -subj "/CN=giveaname" -key cert_key.pem -out cert.csr
	echo "subjectAltName=DNS:your-dns.record,IP:257.10.10.1" >> extfile.cnf
	echo extendedKeyUsage = serverAuth >> extfile.cnf
#### Create the certificate
	openssl x509 -req -sha256 -days 3650 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial

#### Convert crt to pfx certificate
	openssl pkcs12 -export -out new-pfx-cert.pfx -inkey private-key.key -in certificate.crt
