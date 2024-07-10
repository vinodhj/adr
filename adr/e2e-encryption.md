# ADR : End to End Encryption

**Context:**  
To implement end to end encryption for private / anonymous feedbacks.

**Problem:**  
We need to prevent other user from reading private / anonymous feedbacks.

**Considered Options:**

1. **[Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API):** Web Crypto API can be used to derive key pairs and encrypt / decrypt content at client side.
2. **[Node Crypto](https://nodejs.org/api/crypto.html)**: `crypto` package can be used to generate key pairs and encrypt / decrypt contents in a node server.
3. **[AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)** AWS KMS is a dedicated service for storage and retrieval of private / secret keys.
4. **[AWS Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html)** AWS Cognito is a service for managing users and handling token based authentication. There is one feature of custom attribute which can be used to store user level data, which can only be accessed by authorised user.
5. **[AWS Encryption SDK](https://github.com/aws/aws-encryption-sdk-javascript)** This SDK can be used to implement encryption and decryption of data. There are different SDKs for client and server side encryption.

**Discarded Options:**

1. **Node Crypto:** This is not supported by Cloudflare Workers environment, if the server is other than cloudflare workers, then this will work well.
2. **AWS KMS** This is a good and secure option for manager secret / private keys but this is an expensive service, which might not be good option for small projects.
3. **AWS Encryption SDK** This provides similar features like `crypto`, which also require separate system for managing keys. Since this is not a native node js package, so there will be some extra steps of installation and setup, which might not be a good option if `crypto` is sufficient for our use case.

**Decision:**  
We'll use the following services with the respective reasons:

1. **Web Crypto API**

   - The Cloudflare Workers runtime supports this API, unlike Node Crypto which does not work well with Cloudflare Workers.

2. **AWS Congnito**

   - The custom attribute feature can be used to create custom field for private / secret key.
   - Like other user attributes, this attribute can only be accessed by authenticated user.
   - Since, we are already using this service for managing user authentication and authorisation, this will be a good option to manage secret keys.

**Consequences:**

- Team members will need to familiarize themselves with the APIs of `Web Crypto API`.
- Team members will need to familiarize themselves with AWS Cognito Custom Attribute feature.
- Team members will need to familiarise themselves with the concepts of data encoding formats like 'utf-8' and 'base64', 'uint8'.

**Involved Steps for Web Crypto API**
1. Configuration of AWS Cognito for custom attributes. Refer [this doc](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html#user-pool-settings-custom-attributes) for detailed steps.

2. Generation of public / private key pair using `Web Crypto API`
   ```
      // Generate Keys
      const keyPair = await crypto.subtle.generateKey(
      {
         name: "RSA-OAEP",
         modulusLength: 2048,
         publicExponent: new Uint8Array([0x01, 0x00, 0x01]),
         hash: "SHA-256",
      },
      true,
      ["encrypt", "decrypt"]
      );

      // Export public key in PEM format
      const exportedPublicKey = await crypto.subtle.exportKey("spki", keyPair.publicKey);
      const publicKeyPEM = `-----BEGIN PUBLIC KEY-----\n${arrayBufferToBase64String(
      exportedPublicKey
      )}\n-----END PUBLIC KEY-----`;

      // Export private key in PEM format
      const exportedPrivateKey = await crypto.subtle.exportKey("pkcs8", keyPair.privateKey);
      const privateKeyPEM = `-----BEGIN PRIVATE KEY-----\n${arrayBufferToBase64String(
      exportedPrivateKey
      )}\n-----END PRIVATE KEY-----`;

      // Convert Array Buffer to base64String
      const  arrayBufferToBase64String = (buffer: ArrayBuffer): string => {
         let binary = "";
         const bytes = new Uint8Array(buffer);
         for (let i = 0; i < bytes.byteLength; i++) {
            binary += String.fromCharCode(bytes[i]);
         }
         return btoa(binary);
      }   
   ```

3. Storage of private / public key while creating user.

   ```
   import { CognitoIdentityProviderClient, SignUpCommand} from '@aws-sdk/client-cognito-identity-provider';

   const config = {
   region: '<region>',
   credentials: {
      accessKeyId: '<AccessKeyId>',
      secretAccessKey: '<SecretAccessKey>',
   },
   };

   const client = new CognitoIdentityProviderClient(config);

   const command = new SignUpCommand({
   ClientId: '<ClientId>',
   Username: <Username>,
   Password: <Password>,
   UserAttributes: [{
      Name: 'custom:<AttributeName>',  // Needs to be prefixed using 'custom:'
      Value: privateKey  // generated private key
      }],
   });

   await client.send(command);

   // store public key in database or any other place
   // which can be accessed by everyone

4. Encryption of data

```
   // Import and convert public key from PEM format to Buffer
   const importPublicKey = async (publicKeyPem: string) => {
   try {
      // Clean up the PEM format
      const publicKeyBase64Cleaned = publicKeyPem
         .replace("-----BEGIN PUBLIC KEY-----", "")
         .replace("-----END PUBLIC KEY-----", "")
         .replace(/\n/g, "")
         .trim();

      // Decode the Base64 string directly into a Uint8Array
      const publicKeyBinary = Uint8Array.from(atob(publicKeyBase64Cleaned), c => c.charCodeAt(0));

      // Import the public key
      const publicKey = await crypto.subtle.importKey(
         "spki",
         publicKeyBinary,
         {
         name: "RSA-OAEP",
         hash: { name: "SHA-256" }
         },
         true,
         ["encrypt"]
      );

      return publicKey;
   } catch (error) {
      console.error("Error importing public key:", error);
      throw error;
   }
   }

   //Encrypt
   const encrypt = async (data, publicKey) => {
   const encryptedData = await crypto.subtle.encrypt(
      {
         name: "RSA-OAEP"
      },
      publicKey,
      new TextEncoder().encode(data)
   );
   return encryptedData;
   }
```

5. Reading private key from AWS Cognito (should be done just before decryption).

   ```
   import { CognitoIdentityProviderClient, GetUserCommand} from '@aws-sdk/client-cognito-identity-provider';

   const config = {
   region: '<region>',
   credentials: {
      accessKeyId: '<AccessKeyId>',
      secretAccessKey: '<SecretAccessKey>',
   },
   };

   const client = new CognitoIdentityProviderClient(config);


   const command = new GetUserCommand({
      AccessToken: token // access_token of the authenticated user
   });

   const res = await client.send(command);

   const private_key = res.UserAttributes.find(f => f.Name == "custom:<AttributeName>").Value
   ```

6. Decryption of Data
   ```
   // Import and conver private key from PEM format to Buffer
   const importPrivateKey = async (privateKeyPem: string) => {
   try {
      // Clean up the PEM format
      const privateKeyBase64Cleaned = privateKeyPem
         .replace("-----BEGIN PRIVATE KEY-----", "")
         .replace("-----END PRIVATE KEY-----", "")
         .replace(/\n/g, "")
         .trim();

      // Decode the Base64 string directly into a Uint8Array
      const privateKeyBinary = Uint8Array.from(atob(privateKeyBase64Cleaned), c => c.charCodeAt(0));

      // Import the private key
      const privateKey = await crypto.subtle.importKey(
         "pkcs8",
         privateKeyBinary,
         {
         name: "RSA-OAEP",
         hash: { name: "SHA-256" }
         },
         true,
         ["decrypt"]
      );

      return privateKey;
   } catch (error) {
      console.error("Error importing private key:", error);
      throw error;
   }
   }

   //Decrypt

   const decrypt = async (encryptedData, privateKey) => {
   const decryptedData = await crypto.subtle.decrypt(
      {
         name: "RSA-OAEP"
      },
      privateKey,
      encryptedData
   );
   return new TextDecoder().decode(decryptedData);
   }
   ```

7. Encrypting and Decrypting Data
   ```
      const encryptAndDecrypt = async (data, publicKeyPem, privateKeyPem) => {
      // Convert PEM-formatted keys to CryptoKey objects
      const publicKey = await importPublicKey(publicKeyPem);
      const privateKey = await importPrivateKey(privateKeyPem);

      // Encrypt the data using the public key
      const encryptedData = await encrypt(data, publicKey);

      // Decrypt the encrypted data using the private key
      const decryptedData = await decrypt(encryptedData, privateKey);
      console.log({encryptedData, decryptedData})

      return decryptedData;
      }
   ```



**Involved Steps When Using Node Crypto (If environment Supports it):**

1. Configuration of AWS Cognito for custom attributes. Refer [this doc](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html#user-pool-settings-custom-attributes) for detailed steps.

2. Generation of public / private key pair using `crypto`

   ```
   import { generateKeyPairSync } from 'crypto';

   const { privateKey, publicKey} = generateKeyPairSync('rsa', {
      modulusLength: 2048,
      publicKeyEncoding: {
      type: 'spki',  // public key encoding type
      format: 'pem', // public key encoding format
      },
      privateKeyEncoding: {
      type: 'pkcs8', // private key encoding type
      format: 'pem'. // private key encoding format
      },
   });
   ```

3. Storage of private / public key while creating user.

   ```
   import { CognitoIdentityProviderClient, SignUpCommand} from '@aws-sdk/client-cognito-identity-provider';

   const config = {
   region: '<region>',
   credentials: {
      accessKeyId: '<AccessKeyId>',
      secretAccessKey: '<SecretAccessKey>',
   },
   };

   const client = new CognitoIdentityProviderClient(config);

   const command = new SignUpCommand({
   ClientId: '<ClientId>',
   Username: <Username>,
   Password: <Password>,
   UserAttributes: [{
      Name: 'custom:<AttributeName>',  // Needs to be prefixed using 'custom:'
      Value: privateKey  // generated private key
      }],
   });

   await client.send(command);

   // store public key in database or any other place
   // which can be accessed by everyone
   ```

4. Encryption of data using `crypto`.

   ```
   import { publicEncrypt, constants } from 'crypto';

   const encrypted = publicEncrypt(
   {
         key: public_key, // saved public key of the user
         padding: constants.RSA_PKCS1_OAEP_PADDING, // Padding scheme
         oaepHash: 'sha256' // Hash algorithm
   },
   Buffer.from(data, 'utf8') // data is in 'utf-8' format
   );

   // 'base64' format should be used as encrypted data encoding
   const encrypted_data = encrypted.toString('base64');
   ```

5. Reading private key from AWS Cognito (should be done just before decryption).

   ```
   import { CognitoIdentityProviderClient, GetUserCommand} from '@aws-sdk/client-cognito-identity-provider';

   const config = {
   region: '<region>',
   credentials: {
      accessKeyId: '<AccessKeyId>',
      secretAccessKey: '<SecretAccessKey>',
   },
   };

   const client = new CognitoIdentityProviderClient(config);


   const command = new GetUserCommand({
      AccessToken: token // access_token of the authenticated user
   });

   const res = await client.send(command);

   const private_key = res.UserAttributes.find(f => f.Name == "custom:<AttributeName>").Value
   ```

6. Decryption of data using `crypto`.

   ```
   import { publicEncrypt, constants } from 'crypto';

   const decrypted = privateDecrypt(
      {
         key: private_key, // private key from AWS cognito
         padding: constants.RSA_PKCS1_OAEP_PADDING, // Padding scheme used while encrypting
         oaepHash: 'sha256' // Hash algorithm
      },
      Buffer.from(encrypted_data, 'base64') // encrypted_data is in 'base64' format
   );

   // data was in 'utf-8' format
   const data = decrypted.toString('utf-8');
   ```
