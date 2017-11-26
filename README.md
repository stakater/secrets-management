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
