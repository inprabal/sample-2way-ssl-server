
### Generate a Self Signed Certificate for server

support link >>> https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309  

>>> path >>> src/main/resources  

Use Gitbash OpenSSL

1.Create Root Key  
openssl genrsa -des3 -out rootCA.key 4096  

2.Create and self sign the Root Certificate  
>>Here we used our root key to create the root certificate that needs to be distributed in all the computers that have to trust us. Install the certificate in Truststore

openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt

3.Create the certificate key for localhost  
openssl genrsa -out server-app.key 2048

4.Create the signing (csr)  
openssl req -new -key server-app.key -out server-app.csr

5.Verify the csr's content  
openssl req -in server-app.csr -noout -text

6.Generate the certificate using the mydomain csr and key along with the CA Root key  
openssl x509 -req -in server-app.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out server-app.crt -days 500 -sha256

7.Verify the certificate's content  
openssl x509 -in server-app.crt -text -noout

8.Convert server-app.key to server-app.p12  
openssl pkcs12 -export -out server-app.p12 -inkey server-app.key -in server-app.crt -name server-app -certfile rootCA.crt

9.Convert to jks format  
keytool -importkeystore -srckeystore server-app.p12 -srcstoretype PKCS12 -destkeystore server-app.jks  -deststoretype JKS -srcstorepass server-app -deststorepass server-app -srcalias 1 -destalias server-app -srckeypass server-app -destkeypass server-app -noprompt

### Generate Client Certificate for authentication  

1. Create self signed certificate  
Follow the same above steps to generate Client certificate PKCS12 file and install in client machine.  

3. Import client-app.cer to cacerts and rootCA.crt to cacerts  
keytool -import -alias client-app -file client-app.cer -keystore cacerts  

### To build  

mvn clean package spring-boot:repackage  

To run use eithe command  
mvn clean spring-boot:run  
java -jar target/sample-2way-ssl-server-0.0.1-SNAPSHOT.jar  

### Access the site  

https://localhost:8443/server-app/data

