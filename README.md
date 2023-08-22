# All About openssl
## Cheatsheet for OpenSSL
### Step 1 : Download openssl 
### Step 2 : validate installation :
	openssl version -a
### Generate CA
	openssl genrsa -out ca-key.pem 4096
	openssl req -new -x509 -sha256 -days 6300 -key ca-key.pem -out ca.pem
#### Optional Stage: View Certificate's Content
	openssl x509 -in ca.pem -text
	openssl x509 -in ca.pem -purpose -noout -text

### Install the CA Cert as a trusted root CA
#### Assuming the path to your generated CA certificate as `C:\certs\ca.pem`, run: powershell
	Import-Certificate -FilePath "C:\certs\ca.pem" -CertStoreLocation Cert:\LocalMachine\Root
#### OR In Command Prompt, run:
	certutil.exe -addstore root C:\certs\ca.pem

### Generate Certificate
#### Create a Certificate Signing Request (CSR)
	openssl req -new -sha256 -subj "/CN=hostname" -key ca-key.pem -out cert.csr
	echo subjectAltName=DNS:your-dns.record,IP:257.10.10.1 >> extfile.cnf
	echo extendedKeyUsage = serverAuth >> extfile.cnf
#### Create the certificate
	openssl x509 -req -sha256 -days 3650 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial

#### Convert crt to pfx certificate
	openssl pkcs12 -export -out new-pfx-cert.pfx -inkey private-key.key -in certificate.crt
#### Convert pem to pfx certificate
	openssl pkcs12 -inkey ca-key.pem -in cert.pem -export -out cert.pfx
#### Convert pem to pk12
	openssl pkcs12 -export -inkey ca-key.pem -in cert.pem -name localhost -out keystore.p12
#### Convert pk12 to jks usin keytool
	keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore keystore.jks

#### Import the CA-signed certificate to the keystone 
	keytool -import -trustcacerts -alias server-crt -file server.crt -keystore keystore.jks
