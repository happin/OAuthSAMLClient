## OAuthSAMLClient

Sample app that demonstrates the flow to get an OAuth Bearer token based on a SAML2.0 assertion for an SAP Cloud for customer tenant. The token obtained can be used for SSO while invoking OData services offered by SAP Cloud for customer tenant.

### Pre-requisites:
In your SAP Cloud for Customer tenant please register an:
* OAuth IDP (Identity provider) - Note, if you are generating your own SAML signing certificates as given below using OpenSSL, please upload the __samlidp_selfsigned.cer__ as the primary signing certificate when registering the OAuth IDP in C4C.
* OAuth Client

The above mentioned activities can be performed in the Administration work-center in your SAP Cloud for customer tenant. 


In order for the sample to run the following things need to be done:
* Add a JKS key-store to the project (e.g. in the current example the key-store is called as venkyvb.jks).
* Adjust the settings in the **settings.properties** file as follows:
  * CLIENT_ID = valid client ID based on the OAuth client registration mentioned above
  * CLIENT_SECRET = valid client secret associated with the OAuth client registration
  * ISSUER = your app which is registered as an OAuth IDP in your SAP Cloud for Customer tenant
  * NAME_ID = named user in SAP Cloud for customer tenant for whom the OAuth token is being requested (note that in real scenarios this would be determined based on the current logged in user in your app)
  * ENTITY_ID = Tenant URL without the protocol (HTTPS://)
  * TOKEN_SERVICE_URL = https://your_tenant_url/sap/bc/sec/oauth2/token
  * KEY_STORE_PASS = JKS keystore password
  * ...

The token obtained in this step needs to be added as a part of the "Authorization" header for the OData calls. The header should looke like:
* Header name = "Authorization"
* Value = "Bearer " + access_token

Note that this is just an illustrative sample.
Happy coding !!


##### How to use OpenSSL to create the JKS key-store

In case if you are interested, this section outlines how you can use OpenSSL to generate the JKS key-store that would be used by the OAuth client to sign the generated SAML assertions. Note that this example creates a keystore with name __samlidp_keystore.jks__, with the key alias as __samlidp_cert__.

1) Generate signing key for the SAML IDP
```
openssl genrsa -aes256 -out samlidp.key 2048
```

2) Generate cert request for CA
```
openssl req -x509 -sha256 -new -key samlidp.key -out samlidp.csr
```

3) Self sign the certificate (validity of 10 years !!)
```
openssl x509 -sha256 -days 3652 -in samlidp.csr -signkey samlidp.key -out samlidp_selfsigned.cer
```

4) Create pkcs12 keystore
```
openssl pkcs12 -export -name samlidp_cert -in samlidp_selfsigned.cer -inkey samlidp.key -out samlidp_keystore.p12
```

5) Convert pkcs12 into JKS keystore
```
keytool -importkeystore -destkeystore samlidp_keystore.jks -srckeystore samlidp_keystore.p12 -srcstoretype pkcs12 -alias samlidp_cert
```

6) Verify if everything is correct (using the java keytool utility)
```
keytool -list -v -keystore samlidp_keystore.jks
```
