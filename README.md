# secrets-management
how to manage secrets?

## Tools

- vault

## How?

With the Kubernetes auth backend you can easily get your pods Vault tokens which can then be used to access secrets.

[Vault - Auth Backend: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

The Kubernetes auth backend can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

### Basic Concepts of Vault?

[Basic Concepts](https://www.vaultproject.io/docs/concepts/index.html)

### Authentication

Authentication in Vault is the process by which user or machine supplied information is verified against an internal or external system. Before a client can interact with Vault, it must authenticate against an authentication backend. Upon authentication, a token is generated. This token is conceptually similar to a session ID on a website. The token may have attached policy, which is mapped at authentication time.

Vault supports a number of authentication backends. Some backends are targeted toward users while others are targeted toward machines. 

Only one authentication is required to gain access to Vault, and it is not currently possible to force a user through multiple authentication backends to gain access, although some backends do support MFA.

---

#### Step 1 : Authenticate to Vault as User

[Authenticate to Vault using GitHub](https://www.vaultproject.io/docs/auth/github.html)

The GitHub auth backend can be used to authenticate with Vault using a GitHub personal access token. This method of authentication is most useful for humans: operators or developers using Vault directly via the CLI.

**N.B.:** Vault does not support an OAuth workflow to generate GitHub tokens, so does not act as a GitHub application. As a result, this backend uses personal access tokens. An important consequence is that any valid GitHub access token with the read:org scope can be used for authentication. If such a token is stolen from a third party service, and the attacker is able to make network calls to Vault, they will be able to log in as the user that generated the access token. **When using this backend it is a good idea to ensure that access to Vault is restricted at a network level rather than public.**

#### Step 2 : Create a secret in Vault


#### Step 3 : Authenticate to Vault as Service (Pod)

[Auth Backend: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)



---

#### Machine authentication to Vault?



#### Human authentication to Vault?


#### Tokens

It is important to understand that authentication works by verifying your identity and then generating a token to associate with that identity.

For example, even though you may authenticate using something like GitHub, Vault generates a unique access token for you to use for future requests. The CLI automatically attaches this token to requests, but if you're using the API you'll have to do this manually.

#### 


