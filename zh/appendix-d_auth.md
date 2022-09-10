# Appendix D. Security, Authentication, and Authorization in Pulsar

In this book, all of the code snippets share two qualities:

- There is no encryption.
- There is no authentication or authorization.

While operating a real cluster requires good authorization and authentication practices, it was not a wholly necessary requirement to enable these features for pedagogy. I thought it would be appropriate to include some details about security in Pulsar in an appendix where I could introduce some new topics in an environment that was isolated from the rest of our learning. In this appendix you’ll learn about:

- Encryption in transit
- Encryption at rest
- Authentication
- Authorization

# Encryption in Transit

Encryption in transit is ensuring that data traveling over the internet is encrypted. This means that if the data were collected in transit, it could not be read by the interceptor (see [Figure D-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#encryption_in_transit_in_the_pulsar_top)). Encryption in transit is enabled by encrypting the messages before sending them over the wire. In Pulsar you can do that with the following:

```
PulsarClient pulsarClient = PulsarClient.builder()
	.serviceUrl("pulsar://localhost:6650")
	.build();

Producer producer = pulsarClient.newProducer()
                .topic("persistent://my-tenant/my-ns/my-topic")
                .addEncryptionKey("myappkey")
                .cryptoKeyReader(new RawFileKeyReader("test_ecdsa_pubkey.pem",
                  "test_ecdsa_privkey.pem"))
                .create();
```

It uses a public/private key pair for the encryption.

![Encryption in transit in the Pulsar topology uses public/private key pair for the encryption.](../img/mapu_ad01.png)

*Figure D-1. Encryption in transit in the Pulsar topology uses public/private key pairs for the encryption.*

# Encryption at Rest

In Pulsar, we want to encrypt the data in transit and at rest so that only someone with the correct private keys can decrypt messages. In Pulsar, if the data is encrypted by the client, the broker will store the message in BookKeeper encrypted as well. [Figure D-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#trust_certificates_are_used_to_encrypt) provides an illustration of TLS encryption on messages being published and consumed.

To ensure that data is encrypted with TLS, we can create a client that uses a certificate:

```
import org.apache.pulsar.client.api.PulsarClient;

PulsarClient client = PulsarClient.builder()
    .serviceUrl("pulsar+ssl://broker.example.com:6651/")
    .enableTls(true)
    .tlsTrustCertsFilePath("/path/to/ca.cert.pem")
    .enableTlsHostnameVerification(false)
    .allowTlsInsecureConnection(false)
    .build();
```

![Trust certificates are used to encrypt messages.](../img/mapu_ad02.png)

*Figure D-2. Trust certificates are used to encrypt messages.*

# Authentication

Authentication is the process of asking an entity to prove they are who they say they are via credentials. Authentication can take on many forms, including the following:

- Username/password authentication
- Key authentication
- Alternative factor authentication
- SMS authentication

In Pulsar, authentication is used to ensure that clients are who they say they are, and that identity is associated with their authorization. [Figure D-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#in_token-based_authenticationcomma_prod) shows how token-based authentication works, as producers and consumers share their tokens to authenticate with the cluster.

![In token-based authentication, producers and consumers share their tokens to authenticate with the cluster.](../img/mapu_ad03.png)

*Figure D-3. In token-based authentication, producers and consumers share their tokens to authenticate with the cluster.*



Here is an example of a client using token-based authentication:

```
PulsarClient client = PulsarClient.builder()
    .serviceUrl("pulsar://broker.example.com:6650/")
    .authentication(

AuthenticationFactory.token("eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJKb2UifQ.ipevRNuRP6H
flG8cFKnmUPtypruRC4fb1DWtoLL62SY")
    .build();
```

The client uses JSON Web Tokens (JWT) to authenticate with the broker.

# Authorization

Authorization is the privilege of performing actions on given resources. In Pulsar, authorization is used to ensure that an entity can perform actions like creating topics, administering to namespaces, and creating geo-replications. [Figure D-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app04.html#superusers_can_create_clusterscomma_geo) depicts the authorization model in Pulsar. At the top are superusers who can create clusters and geo-replication policies. They can also create tenant admins. Tenant admins can perform actions like creating and managing namespace and tenant policies as well as granting permissions to clients. Clients can create consumers and producers and manage their configurations.

![Superusers can create clusters, geo-replication policies, and tenant admins. Clients can create consumers and producers and manage their configurations.](../img/mapu_ad04.png)

*Figure D-4. Superusers can create clusters, geo-replication policies, and tenant admins. Clients can create consumers and producers and manage their configurations.*

# Summary

Authentication and authorization are critical in making Pulsar safe and usable by large organizations. It organizes the cluster users and ensures that data is safe in transit and at rest. I’m not a security engineer by trade, and every time I get involved with encryption, authentication, and authorization, I get overwhelmed and a little uncertain about my skills. I think many UX improvements can be made in this space to make authentication and authorization much easier. I should not have to generate any certificates.