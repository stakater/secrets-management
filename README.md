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

## Database credentials tend to be static

When it comes to databases, the regular workflow of getting credentials applying for a database is asking some operator or a self-service tool to give you credentials so your application can log into the database. At this point, credentials are considered static. Credentials get usually changed in case the database is migrated or if there’s a security breach.

There’s one caveat: Long-lived credentials are a good target for leakage. Leaked credentials can give access to an unintended party. A few databases implement restrictions on source hosts. In some cases, a database user can be restricted to a group of hosts. That restriction prevents access from other hosts. Still, every user and process that has access to a permitted machine can use the leaked credentials. But how do you discover that leakage? You might find the leak if your data was leaked to the public or the internet but that’s not always the case. For other cases, the unintended party may read or change your data, and it’s fairly sure the leak remains undiscovered for quite a while.

Let’s make credentials short-lived.

## Tools

### 1. Vault

Vault is secret store software. It can be used to safely store and manage credentials. I think that two things distinguish Vault from the rest of the crowd:

- It is designed to be run in the cloud.
- It has the concept of managed backends. A backend can be anything that requires credentials (such as MySQL). Vault will create and rotate credentials for any managed backend.

Managed credentials make Vault interesting for those scenarios where high automation is required, while at the same time strict security requirements on credential management need to be enforced

Doing encryption right is tough, managing secrets is even harder if doing it yourself. Vault addresses exactly these issues. It helps to address the chicken-egg problem and it comes with encryption. Vault is a service to manage secrets. It provides an API that gives access to secrets based on policies. Any user of the API needs to authenticate and only sees the secrets for which he is authorized. Vault encrypts data using 256-bit AES with GCM. It can store data in various backends (files, Amazon DynamoDB, Consul, etcd and much more). The other key aspect is that Vault never stores a key in a persistent location. Starting/restarting Vault always requires one or more operators to unseal Vault. 

### 2. Vault Sidekick

[Vault Sidekick](https://github.com/UKHomeOffice/vault-sidekick)

Vault Sidekick is a add-on container which can be used as a generic entry-point for interacting with Hashicorp Vault service, retrieving secrets (both static and dynamic) and PKI certs. The sidekick will take care of renewal's and extension of leases for you and renew the credentials in the specified format for you.

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

[Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

A service account provides an identity for processes that run in a Pod.

When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).

[Auth Backend: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

The Kubernetes auth backend can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

[Spring Vault Kubernetes Authentication](https://github.com/spring-projects/spring-vault/blob/master/src/main/asciidoc/reference/authentication.adoc#kubernetes-authentication)

Vault supports since 0.8.3 kubernetes-based authentication using Kubernetes tokens.

Using Kubernetes authentication requires a Kubernetes Service Account Token, usually mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`. The file contains the token which is read and sent to Vault. Vault verifies its validity using Kubernets' API during login.

Configuring Kubernetes authentication requires at least the role name to be provided:

```
@Configuration
class AppConfig extends AbstractVaultConfiguration {

    // …

    @Override
    public ClientAuthentication clientAuthentication() {

        KubernetesAuthenticationOptions options = KubernetesAuthenticationOptions.builder()
                .role(…).build();

        return new KubernetesAuthentication(options, restOperations());
    }

    // …
}
```

You can configure the authentication via `KubernetesAuthenticationOptions`.

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

---

## Secrets with Kubernetes

[Deploying Secrets](https://github.com/UKHomeOffice/application-container-platform/blob/master/developer-docs/platform_introduction.md#deploying-secrets)

## Deploying secrets

Your application is likely to have some parameters that are essential to the security of the application - for example API tokens and DB passwords. These should be stored as Kubernetes secrets to enable your application to read them. This process is described in the following guide.

See [official docs](http://kubernetes.io/docs/user-guide/secrets/#creating-a-secret-using-kubectl-create-secret) for more complete documentation; what follows is a very abridged version.

### Generate a strong secret

When generating passwords you should use the following code to ensure you are generating strong passwords.

```bash
$ LC_CTYPE=C tr -dc "[:print:]" < /dev/urandom | head -c 32
```

### Create a kubernetes secret

Create a file called **example-secret.yaml** with the following content below.
In this example, when deployed it will create a kubernetes secret called `my-secret`. Feel free to replace the name `my-secret` with something else, especially if you are working in a group and going through this exercise for the Developer Induction. If you don't you will most likely overwrite each others secret when deploying to Kubernetes:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  supersecret: {{.MY_SECRET}}
```

> *Please note* that you shouldn't commit passwords or sensitive information in your
> repository. We suggest you use environment variables to load secrets into the
> yaml file.

Secrets are passed to kubernetes as base64 encoded strings. To encode or decode a base64 string use the following commands:

```bash
$ echo -n "yay it is a secret" | base64
$ echo eWF5IGl0IGlzIGEgc2VjcmV0 | base64 -D
```

Now let's deploy our secret to Kubernetes:

```bash
$ export MY_SECRET=$(echo -n "replace this secret with something more exciting please!" | base64)
$ kd --file example-secret.yaml
```

### See and edit the stored secrets

You can retrieve the secret with:

```bash
$ kubectl get secrets
$ kubectl describe secret <my-secret>
```

If you wish to edit secrets already loaded in to Kubernetes you can do so by downloading and reapplying the manifest. You can download the secrets as a Yaml file with:

```bash
$ kubectl get secret <my-secret> -o yaml > example-secrets.yaml
```

You can edit the content of  `example-secrets.yaml`, but remember: values are base64 encoded. If you wish to inspect or add a new entry, you need to decode or encode that value.

Once you're done with the changes, you can reapply all the secrets with:

```bash
$ kubectl apply -f example-secrets.yaml
```

> Please note that it's possible to append a key value pair to an existing secret. You can however download the secret's manifest and reapply the changes as explained above.

### Use the secrets

You can mount secrets into your application using either mounted volumes or by using them as environment variables.

The below example shows a deployment that does both. However for this challenge please update your deployment in acp-hello-world to use your secret as an environment variable called MYSUPERSECRET.

<details>
<summary>**This yaml is an example! Please do not copy and paste, just use it as a guide to modify your own deployment.yaml!**</summary>

```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: induction-hello-world
spec:
  volumes:
    - name: "secretstest"
      secret:
        secretName: mysecret
  containers:
    - image: nginx:1.9.6
      name: awebserver
      volumeMounts:
        - mountPath: "/tmp/mysec"
          name: "secretstest"
      env:
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
                name: my-secret
                key: dbpass
        - name: USERNAME
          valueFrom:
            secretKeyRef:
                name: my-secret
                key: dbuser
        - name: HOST
          valueFrom:
            secretKeyRef:
                name: my-secret
                key: dbhost
```
</details>

For your own deployment.yaml file you should have an env section in the appropriate place that looks similar to this. The `name` and `key` fields should be the same:

```yaml
env:
  - name: MYSUPERSECRET
    valueFrom:
      secretKeyRef:
          name: <what you have named your secret>
          key: supersecret
```

Once you've updated your deployment file to set the `MYSUPERSECRET` environment variable using the kubernetes secret, you will need to redeploy it:

```bash
$ APP_NAME=tgxu172 \
  APP_VERSION=v1 \
  kd --file kube-templated/deployment.yaml
```

Now when you navigate to https://tgxu172.notprod.homeoffice.gov.uk/ you should see your secret outputted as part of the message.

### Debug with secrets

Sometimes your app doesn't want to talk to an API or a DB and you've stored the credentials or just the details of that in secret.

The following approaches can be used to validate that your secret is set correctly

```bash
$ kubectl exec -ti my-pod -c my-container -- mysql -h\$DBHOST -u\$DBUSER -p\$DBPASS
## or
$ kubectl exec -ti my-pod -c my-container -- openssl verify /secrets/certificate.pem
## or
$ kubectl exec -ti my-pod -c my-container bash
## and you'll naturally have all the environment variables set and volumes mounted.
## however we recommend against outputing them to the console e.g. echo $DBHOST
## instead if you want to assert a variable is set correctly use
$ [[ -z $DBHOST ]]; echo $?
## if it returns 1 then the variable is set.
```

---

## References

- https://spring.io/blog/2016/06/24/managing-secrets-with-vault
- https://blog.openshift.com/managing-secrets-openshift-vault-integration/
- https://spring.io/blog/2016/08/15/managing-your-database-secrets-with-vault
