---
layout: post
title:  "Enabling Client Assertion on Kong Gateway"
date:   2021-07-18 10:17:00 +0800
categories: KongGateway
---
# Enabling Client Assertion on Kong Gateway

This setup assumes Kong is running without a database.

# Summary

The following table outlines the task to be carried out.

|#|Activities|Description|
|-|-|-|
|1|Prepare Consumer Profile|This exercise will be based on the Client Assertion when authenticating against the Kong Gateway. Hence an asymmetric key will be generated.|
|2|Add service to the Kong Gateway and enable Client Assertion|Register an API service and add the customer profile to allow access. The Kong Gateway setup will be based on a configuration file instead of a database.
|3|Verify Kong Gateway Connectivity and service configuration|Test the Kong Gateway is reachable and verify the setup is as expected.
|4|Develop Java Application to consume the API service||



## Kong configuration

### Prepare Consumer Profile
1. Generate asymmetric key for a client
    ```
    openssl genrsa -out private.pem 2048
    openssl rsa -in private.pem -outform PEM -pubout -out public.pem
    ```
2. Convert key to a PKCS#8 format
    ```
    openssl pkcs8 -topk8 -inform PEM -outform DER -in private.pem  -nocrypt > pkcs8.key
    ```
### Add service to the Kong Gateway and enable Client Assertion

1. Convert the public key to a single line for adding to the `kong.yml` file.
    ```
    awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' public.pem 
    ```
    Variable `PUBLIC_KEY` below should refer to this value.
2. Update `kong.conf` file

    |Variable|Description|Example|
    |-|-|-|
    |SERVICE_NAME|Name of the API service|xrp-service|
    |API_HOST_URL|API Endpoint URL|http://localhost:8081|
    |ROUTE_NAME||xrp-route|
    |CONSUMER_NAME|Name of the API consumer|dennis|
    |PUBLIC_KEY|Consumer's public key in PKCS#8 format||
    
   ```
   _format_version: "2.1"
    _transform: true

    services:
    - name: ${SERVICE_NAME}
      url: ${API_HOST_URL}
      routes:
      - name: ${ROUTE_NAME}
        tags:
        - xrp-wallets
        paths:
        - /wallets
        strip_path: false

    consumers:
    - username: ${CONSUMER_NAME}
    
    jwt_secrets:
    - consumer: ${CONSUMER_NAME}
      algorithm: RS256
      rsa_public_key: "${PUBLIC_KEY}"

    plugins:
    - name: rate-limiting
      service: ${SERVICE_NAME}
      config:
        minute: 5
        policy: local
    - name: jwt
      service: ${SERVICE_NAME}
      config:
        secret_is_base64: false
        run_on_preflight: true
   ```

3. Start Kong server
    ```
    kong start -c /etc/kong/kong.conf
    ```

4. Verify Kong Gateway Connectivity	and Service configuration
    ```
    curl http://localhost:8001/services
    ```
    ```json
    {
      "next": null,
      "data": [
        {
          "tls_verify": null,
          "port": 8081,
          "tls_verify_depth": null,
          "tags": null,
          "ca_certificates": null,
          "protocol": "http",
          "connect_timeout": 60000,
          "read_timeout": 60000,
          "path": null,
          "id": "f631da89-605d-53a5-93db-f7cc633371db",
          "host": "172.31.32.165",
          "updated_at": 1626571999,
          "name": "xrp-service",
          "retries": 5,
          "created_at": 1626571999,
          "write_timeout": 60000,
          "client_certificate": null
        }
      ]
    }
    ```

# Develop Java Application to consume the API service
1. Generate maven project in the chosen folder.
    ```
    mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-simple -DarchetypeVersion=1.4
    ```
2. Place the Private Key in PKCS#8 format to the resources file. 
3. Identity the issuer from the Kong Gateway as follows:
    ```
    curl http://localhost:8001/consumers/dennis/jwt
    ```
    
    ```json
    {
        "next": null,
        "data": [
            {
                "id": "1f79898d-5f62-461d-a33e-5b205b0ae249",
                "rsa_public_key": "${PUBLIC_KEY},
                "created_at": 1626571999,
                "tags": null,
                "key": "mTqEnyJeUL09j8ofOa68z5vOtTVj2Pd6",
                "consumer": {
                    "id": "d05f5688-2910-5460-a2c9-ce900630c881"
                },
                "algorithm": "RS256",
                "secret": "JaomgZgIkgKre7fsZZnKpuoEu30u4BSx"
            }
        ]
    }
    ```
3. Create a Java application to generate the JWT.
    |Variable|Description|Example|
    |-|-|-|
    |KEY|Issuer's key acquired from the Kong Gateway|mTqEnyJeUL09j8ofOa68z5vOtTVj2Pd6|

    ```Java
    import java.io.File;
    import java.nio.charset.Charset;
    import java.nio.file.Files;
    import java.security.KeyFactory;
    import java.security.PrivateKey;
    import java.security.spec.KeySpec;
    import java.security.spec.PKCS8EncodedKeySpec;
    import java.util.Base64;
    import java.util.HashMap;
    import java.util.Map;
    import java.util.UUID;
    
    public class APIConsumer {
        public static void main(String[] args) {
            String filePath = APIConsumer.class.getResource("/pkcs8.key1").getPath();
        	File file = new File(filePath);
        
        	String key = new String(Files.readAllBytes(file.toPath()), Charset.defaultCharset());
        
        	String privateKeyPEM = key.replace("-----BEGIN PRIVATE KEY-----", "").replaceAll(System.lineSeparator(), "")
        					.replace("-----END PRIVATE KEY-----", "");
        
        	byte[] encoded = Base64.getDecoder().decode(privateKeyPEM);
        	KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        	KeySpec keySpec = new PKCS8EncodedKeySpec(encoded);
        
        	PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
        
        	Map<String, Object> headers = new HashMap<String, Object>();
        	headers.put("typ", "JWT");
        	headers.put("alg", "RS256");
        			
        	Map<String, Object> claims = new HashMap<String, Object>();
        			
        	claims.put("iat", System.currentTimeMillis() / 1000);
        	claims.put("jti", UUID.randomUUID().toString());
        	claims.put("iss", "${KEY}");
        			
        			
        	String jwt = Jwts.builder().setHeader(headers).setClaims(claims).signWith(privateKey).compact();
        			
        	System.out.println(String.format("JWT> %s", jwt));
	    }
	}
    ```
4. The filesystem structure should be as below.
    ```
       |-src
       |---main
       |-----java
       |-------io
       |---------forest
       |-----------APIConsumer.java
       |-----resources
       |-------pkcs8.key
    ```

# Glossary

|#|Terms|Definition|
|-|-|-|
|1|PKCS#8|Private-Key Information Syntax Standard. Used to carry private certificate keypairs (encrypted or unencrypted). (src: [https://en.wikipedia.org/wiki/PKCS_8](https://en.wikipedia.org/wiki/PKCS_8)|

# Useful Links
* [https://docs.konghq.com/hub/kong-inc/jwt/](https://docs.konghq.com/hub/kong-inc/jwt/)

