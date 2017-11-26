# secrets-management
how to manage secrets?

Credentials are environment dependent configurations that need to be kept secret and should be read only by subjects with a need-to-know.

Passwords, API keys and confidential data fall into the category of secrets. Storing secrets the secure way is a challenge with limiting access and a true secure storage. 

## How do you store Secrets?

Passwords, API keys, secure Tokens, and confidential data fall into the category of secrets.

That’s data which shouldn’t lie around. It mustn’t be available in plaintext in easy to guess locations. In fact, it must not be stored in plaintext in any location.

Sensitive data can be encrypted by using the Spring Cloud Config Server or TomEE.

Encrypted data is one step better than unencrypted. Encryption imposes on the other side the need for decryption on the user side which requires a decryption key to be distributed. Now, where do you put the key? Is the key protected by a passphrase? Where do you put the passphrase? On how many systems do you distribute your key and the passphrase?

As you see, encryption introduces a **chicken-egg problem**. Storing a decryption key gives the application the possibility to decrypt data. It also allows an attack vector. Someone who is not authorized could get access to the decryption key by having access to the machine. That person can decrypt data which is decryptable by this key. The key is static so a leaked key requires the change of keys. Data needs to be re-encrypted and credentials need to be changed. It’s not possible to discover such leakage with online measure because data can be decrypted offline once it was obtained.

One approach is putting the key in a hard to guess location before the application starts and wipe the key once it was read to memory. The time in which the key is available is shortened. The attack time-frame is reduced, but still the key was there. Wiping the key works only for one application startup. Containers and microservices in the Cloud are known to be restarted once they crashed. A restart of the application is no longer possible as the key is gone.

## Tools

### Vault

Vault is secret store software. It can be used to safely store and manage credentials. I think that two things distinguish Vault from the rest of the crowd:

- It is designed to be run in the cloud.
- It has the concept of managed backends. A backend can be anything that requires credentials (such as MySQL). Vault will create and rotate credentials for any managed backend.

Managed credentials make Vault interesting for those scenarios where high automation is required, while at the same time strict security requirements on credential management need to be enforced

Doing encryption right is tough, managing secrets is even harder if doing it yourself. Vault addresses exactly these issues. It helps to address the chicken-egg problem and it comes with encryption. Vault is a service to manage secrets. It provides an API that gives access to secrets based on policies. Any user of the API needs to authenticate and only sees the secrets for which he is authorized. Vault encrypts data using 256-bit AES with GCM. It can store data in various backends (files, Amazon DynamoDB, Consul, etcd and much more). The other key aspect is that Vault never stores a key in a persistent location. Starting/restarting Vault always requires one or more operators to unseal Vault. 

## How?

With the Kubernetes auth backend you can easily get your pods Vault tokens which can then be used to access secrets.

[Vault - Auth Backend: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

The Kubernetes auth backend can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

---

#### Step 1 : Authenticate to Vault as User

[Authenticate to Vault using GitHub](https://www.vaultproject.io/docs/auth/github.html)

The GitHub auth backend can be used to authenticate with Vault using a GitHub personal access token. This method of authentication is most useful for humans: operators or developers using Vault directly via the CLI.

**N.B.:** Vault does not support an OAuth workflow to generate GitHub tokens, so does not act as a GitHub application. As a result, this backend uses personal access tokens. An important consequence is that any valid GitHub access token with the read:org scope can be used for authentication. If such a token is stolen from a third party service, and the attacker is able to make network calls to Vault, they will be able to log in as the user that generated the access token. **When using this backend it is a good idea to ensure that access to Vault is restricted at a network level rather than public.**

#### Step 2 : Create a secret in Vault


#### Step 3 : Authenticate to Vault as Service (Pod)

[Auth Backend: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

The Kubernetes auth backend can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

##### Security Model

The current authentication model requires providing Vault with a Service Account token, which can be used to make authenticated calls to Kubernetes. This token should not typically be shared, but in order for Kubernetes to be treated as a trusted third party, Vault must validate something that Kubernetes has cryptographically signed and that conveys the identity of the token holder.

---


### Basic Concepts of Vault?

[Basic Concepts](https://www.vaultproject.io/docs/concepts/index.html)

### Authentication

Authentication in Vault is the process by which user or machine supplied information is verified against an internal or external system. Before a client can interact with Vault, it must authenticate against an authentication backend. Upon authentication, a token is generated. This token is conceptually similar to a session ID on a website. The token may have attached policy, which is mapped at authentication time.

Vault supports a number of authentication backends. Some backends are targeted toward users while others are targeted toward machines. 

Only one authentication is required to gain access to Vault, and it is not currently possible to force a user through multiple authentication backends to gain access, although some backends do support MFA.

#### Machine authentication to Vault?



#### Human authentication to Vault?


#### Tokens

It is important to understand that authentication works by verifying your identity and then generating a token to associate with that identity.

For example, even though you may authenticate using something like GitHub, Vault generates a unique access token for you to use for future requests. The CLI automatically attaches this token to requests, but if you're using the API you'll have to do this manually.

#### 


## References

- https://spring.io/blog/2016/06/24/managing-secrets-with-vault
- https://blog.openshift.com/managing-secrets-openshift-vault-integration/
- https://spring.io/blog/2016/08/15/managing-your-database-secrets-with-vault
