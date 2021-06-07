### Crypto Companion Database

#### General Description of the Prototype

Since it is necessary that sensitive data stored has to be secured and private, the crypto companion database (CCDB) is proposed as a parallel system to the blockchain for the encrypted storage. The blockchain will save a hash created from the encrypted sensitive data, and the CCDB will store the sensitive data encrypted together with the hash. The hash will be used to have a connection between the transaction in the blockchain and the data stored in the database.

In order to be complaint with the GDPR, the hash stored in the Blockchain is not going to be created from original but the encrypted data.

The CCDB will encrypt the data with an asymmetric public/private key pair.

This data could only be accessed by the owner, which will have to be authenticated, and the authorized operators allowed by the owner. The authorization is not part of the CCDB as it will be carried by the Blockchain itself, so the component will ask the Blockchain for it.

With this database insertion, deletion and consultation of the information will be possible. The modification process will be a bit more complex as the hash that holds the link between the blockchain and the database will change if the information changes, so if a modification is needed it will be done by deleting the old information and inserting the new one.

Each application in the ecosystem can have its own crypto companion database; therefore data will always be distributed. In order to make it accessible and replicated if wanted, the key pair can be replicated on any system by providing the 12-word mnemonic. To reduce the amount of data held by a single database, the location of specific information can be stated in the blockchain transaction.

Following a figure showing how data can be accessed:

![](RackMultipart20210607-4-1q88eg9_html_4fdb5dc5237ee228.png)

**Figure 20. Access to distributed data.**

#### Components

The components used to create the CCDB are the following:

- Crypto Module
- Companion DB Module

The evolution of the Companion DB Module with the Crypto Module makes a secured database, as the data will be encrypted by an asymmetric key pair, so it will be called **Crypto Companion DB**.

The Crypto Module will be used independently on any type of database (currently only supports MongoDB) and in any software because it provides an API to encrypt/decrypt data. The API of this module is designed as a private API with no access to the internet, so it does not provide any security.

The Companion DB Module has a public API that can be used to save, delete and query data. It also provides an authentication layer in order to secure the users that access the data. This module also provides an authorization layer in order to know if the owner of the data allows an operator or external user to see it. This authorization layer will make use of the Smart Contracts on Blockchain described in the previous sections.

In the following diagram an overview of the components.

![](RackMultipart20210607-4-1q88eg9_html_7facefe76a0e7b06.png)

**Figure 21. Crypto Companion Database Module components.**

#### Crypto Module

This module allows a user to encrypt and decrypt data.

This module has two components:

- The Crypto API, that is in charge of encrypts and decrypts the data with the keys stored in the KeyStore DB.
- The KeyStore DB, that is a MongoDB that holds the key pairs to encrypt and decrypt data by the users.

The Crypto API provides:

- A method to create an asymmetric key pair:

![](RackMultipart20210607-4-1q88eg9_html_3f520ad56d139ce5.png)

The creation of the public/private key pair can be made by providing a 12-word mnemonic, allowing replicating the keys in other applications. It will be useful if the user wants to authorize always with the same public/private key, and also will allow a distributed system to be able to decrypt data in a distributed way.

![](RackMultipart20210607-4-1q88eg9_html_e4de0ef546ead5ff.png)

**Figure 22. Sequence diagram. Enrolment in Crypto Module.**

- A method to encrypt data:

![](RackMultipart20210607-4-1q88eg9_html_b27532bee4d5a4bf.png)

This endpoint will take the private key of the user with the hash provided and encrypt the string with the data in the payload. If the user does not exists it will return the data sent as it is.

![](RackMultipart20210607-4-1q88eg9_html_21af96b5989e8501.png)

**Figure 23. Sequence diagram. Data encryption in Crypto Module.**

- A method to decrypt data:

![](RackMultipart20210607-4-1q88eg9_html_3d30aa22c4a18c89.png)

This endpoint will take the private key of the user with the hash provided and decrypt the string with the data in the payload. If the user does not exists it will return the data sent as it is.

![](RackMultipart20210607-4-1q88eg9_html_dfccaa229031d97a.png)

**Figure 24. Sequence diagram. Data decryption in Crypto Module.**

- A method to delete the keys:

![](RackMultipart20210607-4-1q88eg9_html_bf5b7d7eac1f4141.png)

This endpoint will delete the public/private keys associated with the hash provided.

![](RackMultipart20210607-4-1q88eg9_html_88c53d933a2bbdfd.png)

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

![](RackMultipart20210607-4-1q88eg9_html_7cdd051691b8b7ce.png)

**Figure 26. Authentication API in Crypto Companion Database Module.**

- A method to register:

![](RackMultipart20210607-4-1q88eg9_html_7cdd051691b8b7ce.png)

The registration of a user will also trigger the enrolment on the Crypto Module, so the keys will be created during the registration.

#### Data Management API.

- A method to enrol:

![](RackMultipart20210607-4-1q88eg9_html_175b99d8c8c02408.png)

The creation of the public/private key pair can be made by providing a 12-word mnemonic, allowing replicating the keys in other applications. It will be useful if the user wants to authorize always with the same public/private key, and also will allow a distributed system to be able to decrypt data in a distributed way.

![](RackMultipart20210607-4-1q88eg9_html_ab5aa61cbc057088.png)

**Figure 27. Sequence diagram. Enrolment in CCDB Module.**

- A method to disenrol:

![](RackMultipart20210607-4-1q88eg9_html_d3973eb4c2f423e.png)

This endpoint will delete all data associated with the user along with its public/private keys.

![](RackMultipart20210607-4-1q88eg9_html_b7beb8e1caaded3b.png)

**Figure 28. Sequence diagram. Disenrolment in CCDB Module.**

- A method to read data:

![](RackMultipart20210607-4-1q88eg9_html_4ac5741761904a1.png)

These endpoints will let an owner or an authorized user to read the encrypted data.

![](RackMultipart20210607-4-1q88eg9_html_7d31d6f70bb0436f.png)

**Figure 29. Sequence diagram. Read data in CCDB Module.**

- A method to save data:

![](RackMultipart20210607-4-1q88eg9_html_463cbc29cc4ccce2.png)

This endpoint will let an owner to save encrypted data.

This method has evolved in order to be more compliant with the GDPR.

Before the user should call the blockchain outside and provide a hash in order to link the information between the CCDB and the blockchain. Now, the companion database will take care of the encryption and the hash generation, making it more secure and having a hash in the blockchain that will be generated from encrypted sensitive data, not the raw sensitive data.

![](RackMultipart20210607-4-1q88eg9_html_d3e380560c240c33.png)

**Figure 30. Sequence Diagram. Save data in CCDB Module.**

- A method to delete data:

![](RackMultipart20210607-4-1q88eg9_html_6d402261875a98dc.png)

These endpoints will let the owner of the data to delete it.

![](RackMultipart20210607-4-1q88eg9_html_662a04f983c0840d.png)

**Figure 31. Sequence Diagram. Delete data in CCDB Module.**

- A method to authorize a user:

![](RackMultipart20210607-4-1q88eg9_html_bfc2026eb3f87129.png)

This endpoint will let an owner to authorize another user to decrypt its data.

![](RackMultipart20210607-4-1q88eg9_html_5a5c93df3289dd13.png)

**Figure 32. Sequence diagram. Authorize in CCDB Module.**

- A method to de-authorize a user:

![](RackMultipart20210607-4-1q88eg9_html_51b176e8ebb23be5.png)

This endpoint will let an owner to de-authorize another user to decrypt its data.

![](RackMultipart20210607-4-1q88eg9_html_ccdeb04951ca8e89.png)

**Figure 33. Sequence diagram. Descoritase in CCDB Module.**

- A method to request authorization to a user:

![](RackMultipart20210607-4-1q88eg9_html_d1b03c75503d3e20.png)

This endpoint will let an external user to request authorization to access data to the owner.

![](RackMultipart20210607-4-1q88eg9_html_432454ac420f79f2.png)

**Figure 34. Sequence diagram. Request authorization in CCDB Module.**
