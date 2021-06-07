### Crypto Companion Database

#### General Description of the Prototype

Since it is necessary that sensitive data stored has to be secured and private, the crypto companion database (CCDB) is proposed as a parallel system to the blockchain for the encrypted storage. The blockchain will save a hash created from the encrypted sensitive data, and the CCDB will store the sensitive data encrypted together with the hash. The hash will be used to have a connection between the transaction in the blockchain and the data stored in the database.

In order to be complaint with the GDPR, the hash stored in the Blockchain is not going to be created from original but the encrypted data.

The CCDB will encrypt the data with an asymmetric public/private key pair.

This data could only be accessed by the owner, which will have to be authenticated, and the authorized operators allowed by the owner. The authorization is not part of the CCDB as it will be carried by the Blockchain itself, so the component will ask the Blockchain for it.

With this database insertion, deletion and consultation of the information will be possible. The modification process will be a bit more complex as the hash that holds the link between the blockchain and the database will change if the information changes, so if a modification is needed it will be done by deleting the old information and inserting the new one.

Each application in the ecosystem can have its own crypto companion database; therefore data will always be distributed. In order to make it accessible and replicated if wanted, the key pair can be replicated on any system by providing the 12-word mnemonic. To reduce the amount of data held by a single database, the location of specific information can be stated in the blockchain transaction.

Following a figure showing how data can be accessed:

![image](https://user-images.githubusercontent.com/10829055/121028556-3a364380-c7a8-11eb-96d3-79dc0ebf519a.png)

**Figure 20. Access to distributed data.**

#### Components

The components used to create the CCDB are the following:

- Crypto Module
- Companion DB Module

The evolution of the Companion DB Module with the Crypto Module makes a secured database, as the data will be encrypted by an asymmetric key pair, so it will be called **Crypto Companion DB**.

The Crypto Module will be used independently on any type of database (currently only supports MongoDB) and in any software because it provides an API to encrypt/decrypt data. The API of this module is designed as a private API with no access to the internet, so it does not provide any security.

The Companion DB Module has a public API that can be used to save, delete and query data. It also provides an authentication layer in order to secure the users that access the data. This module also provides an authorization layer in order to know if the owner of the data allows an operator or external user to see it. This authorization layer will make use of the Smart Contracts on Blockchain described in the previous sections.

In the following diagram an overview of the components.

![image](https://user-images.githubusercontent.com/10829055/121028589-402c2480-c7a8-11eb-86cf-8a4871674d84.png)

**Figure 21. Crypto Companion Database Module components.**

#### Crypto Module

This module allows a user to encrypt and decrypt data.

This module has two components:

- The Crypto API, that is in charge of encrypts and decrypts the data with the keys stored in the KeyStore DB.
- The KeyStore DB, that is a MongoDB that holds the key pairs to encrypt and decrypt data by the users.

The Crypto API provides:

- A method to create an asymmetric key pair:

![image](https://user-images.githubusercontent.com/10829055/121028774-5fc34d00-c7a8-11eb-8f39-6f23b3d8cba3.png)

The creation of the public/private key pair can be made by providing a 12-word mnemonic, allowing replicating the keys in other applications. It will be useful if the user wants to authorize always with the same public/private key, and also will allow a distributed system to be able to decrypt data in a distributed way.

![image](https://user-images.githubusercontent.com/10829055/121028796-65209780-c7a8-11eb-9f7b-8af28255d25e.png)

**Figure 22. Sequence diagram. Enrolment in Crypto Module.**

- A method to encrypt data:

![image](https://user-images.githubusercontent.com/10829055/121028806-681b8800-c7a8-11eb-8891-be022d5424b8.png)

This endpoint will take the private key of the user with the hash provided and encrypt the string with the data in the payload. If the user does not exists it will return the data sent as it is.

![image](https://user-images.githubusercontent.com/10829055/121028864-71a4f000-c7a8-11eb-82ac-29a92f9d286d.png)

**Figure 23. Sequence diagram. Data encryption in Crypto Module.**

- A method to decrypt data:

![image](https://user-images.githubusercontent.com/10829055/121028899-79649480-c7a8-11eb-9b3d-714dd200f82c.png)

This endpoint will take the private key of the user with the hash provided and decrypt the string with the data in the payload. If the user does not exists it will return the data sent as it is.

![image](https://user-images.githubusercontent.com/10829055/121028926-7c5f8500-c7a8-11eb-8e92-ed3cd83e971c.png)

**Figure 24. Sequence diagram. Data decryption in Crypto Module.**

- A method to delete the keys:

![image](https://user-images.githubusercontent.com/10829055/121028968-82edfc80-c7a8-11eb-9b4d-d3f208e73494.png)

This endpoint will delete the public/private keys associated with the hash provided.

![image](https://user-images.githubusercontent.com/10829055/121028987-85e8ed00-c7a8-11eb-89de-9bf76d49de38.png)

**Figure 25. Sequence diagram. Disenrolment in Crypto Module.**

The API of this module is intended to be private, so it does not provide any kind of security.

The KeyStore DB will store the public and private keys created by the Crypto API with a hash that will act as an identifier.

So the keys stored in the database will look like:

- hash: 32-64 hexadecimal string identifying the user.
- privateKey: The Private key generated by an asymmetric key algorithm that matches the public key.
- mnemonic: a set of 12-word that is used to create an account into the blockchain and to generate the privateKey.
- blockchainOwnerKeys: Object provided by the creation of a user in the blockchain.

#### Crypto Companion DB Module

This module allows a user to have an authentication system and save data encrypted. It also provides other users with the possibility to read data from a user if authorized.

This module has two components:

- The Companion API, is in charge of authentication and managing all the data providing methods to save, read and delete data in the Companion DB.
- The Companion DB, is a MongoDB that stores the encrypted data.

The Companion DB API provides:

#### Authentication API.

- A set of methods to register, update user information and recover a password.

![image](https://user-images.githubusercontent.com/10829055/121029026-8da89180-c7a8-11eb-999c-237e93f04a1d.png)

**Figure 26. Authentication API in Crypto Companion Database Module.**

- A method to register:

![image](https://user-images.githubusercontent.com/10829055/121029041-913c1880-c7a8-11eb-8827-4c73dda46ade.png)

The registration of a user will also trigger the enrolment on the Crypto Module, so the keys will be created during the registration.

#### Data Management API.

- A method to enrol:

![image](https://user-images.githubusercontent.com/10829055/121029061-94370900-c7a8-11eb-93ef-86fb594a6d3e.png)

The creation of the public/private key pair can be made by providing a 12-word mnemonic, allowing replicating the keys in other applications. It will be useful if the user wants to authorize always with the same public/private key, and also will allow a distributed system to be able to decrypt data in a distributed way.

![image](https://user-images.githubusercontent.com/10829055/121029078-97ca9000-c7a8-11eb-810f-fc335a4accc0.png)

**Figure 27. Sequence diagram. Enrolment in CCDB Module.**

- A method to disenrol:

![image](https://user-images.githubusercontent.com/10829055/121029099-9b5e1700-c7a8-11eb-9f25-871a836b036e.png)

This endpoint will delete all data associated with the user along with its public/private keys.

![image](https://user-images.githubusercontent.com/10829055/121029128-a153f800-c7a8-11eb-963e-000d30d4780c.png)

**Figure 28. Sequence diagram. Disenrolment in CCDB Module.**

- A method to read data:

![image](https://user-images.githubusercontent.com/10829055/121029169-ad3fba00-c7a8-11eb-9a67-eed5286c889e.png)

These endpoints will let an owner or an authorized user to read the encrypted data.

![image](https://user-images.githubusercontent.com/10829055/121029184-b03aaa80-c7a8-11eb-8592-6a8fb87dcecd.png)

**Figure 29. Sequence diagram. Read data in CCDB Module.**

- A method to save data:

![image](https://user-images.githubusercontent.com/10829055/121029207-b4ff5e80-c7a8-11eb-9d9c-98d6e2c64d0a.png)

This endpoint will let an owner to save encrypted data.

This method has evolved in order to be more compliant with the GDPR.

Before the user should call the blockchain outside and provide a hash in order to link the information between the CCDB and the blockchain. Now, the companion database will take care of the encryption and the hash generation, making it more secure and having a hash in the blockchain that will be generated from encrypted sensitive data, not the raw sensitive data.

![image](https://user-images.githubusercontent.com/10829055/121029271-c0eb2080-c7a8-11eb-8327-c88f414df898.png)

**Figure 30. Sequence Diagram. Save data in CCDB Module.**

- A method to delete data:

![image](https://user-images.githubusercontent.com/10829055/121029293-c34d7a80-c7a8-11eb-8d19-dba854af4c0e.png)

These endpoints will let the owner of the data to delete it.

![image](https://user-images.githubusercontent.com/10829055/121029308-c6e10180-c7a8-11eb-9d3e-d6e0a0da68f7.png)

**Figure 31. Sequence Diagram. Delete data in CCDB Module.**

- A method to authorize a user:

![image](https://user-images.githubusercontent.com/10829055/121029329-ca748880-c7a8-11eb-8187-340d5b84d000.png)

This endpoint will let an owner to authorize another user to decrypt its data.

![image](https://user-images.githubusercontent.com/10829055/121029347-cd6f7900-c7a8-11eb-9087-573957df58cf.png)

**Figure 32. Sequence diagram. Authorize in CCDB Module.**

- A method to de-authorize a user:

![image](https://user-images.githubusercontent.com/10829055/121029367-d1030000-c7a8-11eb-8770-f5b81acb0159.png)

This endpoint will let an owner to de-authorize another user to decrypt its data.

![image](https://user-images.githubusercontent.com/10829055/121029379-d3fdf080-c7a8-11eb-8c84-e211a64dfb96.png)

**Figure 33. Sequence diagram. Descoritase in CCDB Module.**

- A method to request authorization to a user:

![image](https://user-images.githubusercontent.com/10829055/121029470-e5df9380-c7a8-11eb-884b-6e773348490c.png)

This endpoint will let an external user to request authorization to access data to the owner.

![image](https://user-images.githubusercontent.com/10829055/121029489-ea0bb100-c7a8-11eb-97b9-79cdc201b64c.png)

**Figure 34. Sequence diagram. Request authorization in CCDB Module.**
