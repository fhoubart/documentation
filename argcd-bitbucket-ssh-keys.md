# Configure ArgoCD to access Bitbucket private repo with SSH keys

Using a private repository with ArgoCD require setting up authentication. You should not use your personal username and password for an application for security reasons. Anyway, if MFA is activated on your account you won't be able to use you personal account with an application.

Bitbucket provides App password for such scenario. You will get a specific password on which you can affect scopes, and use it with your application without MFA. This app password cannot be used to access Bitbucket interface or your account.

App password still have a major drawback: even if you can set scope to restrict the app's permissions, you cannot restrict it to a specific repository.

Bitbucket also provide SSH access, and the ability to set the keys at the repository level and restrict access to this single repository.

# Bitbucket configuration

## Generate a SSH key pair

Use the following command to generate a new key pair on the current directory:

```
ssh-keygen -f bitbucket-ssh-key
```

This will generate two files :

- bitbucket-ssh-key: the private key that need to be kept save
- bitbucket-ssh-key.pub: the public key that will be set on bitbucket

## Configure the public key on Bitbucket

In the repository that you want to connect, go to `Repository settings -> Access keys` and hit **Add key** button. Give a name to your key and copy the content of the public key file in the corresponding field.

## Test the SSH authentication

You may want to test your settup before going into ArgoCD with a manual git clone. You need to tell SSH where to find the ssh private key to authenticate to Bitbucket.

Open (or create if it does not exist) the ~/.ssh/config` file and add the following content:

```
Host bitbucket.org
	HostName bitbucket.org
	IdentityFile /path/to/your/*private*/key
```

Be sure to check the permissions of this file that need to be 600, otherwise ssh will ignore it: `chmod 660 ~/.ssh/config`

You should be able to clone your private repo with SSH without providing any password.


# ArgoCD configuration

Now you have a working ssh key pair, you need to use it to connect ArgoCD to your repository. You can either do it on ArgoCD UI, or via declarative configuration with a Secret that will contain the repo url and the ssh key.

Create the Secret in a `argocb-bitbucket.yml` file:

```
apiVersion: v1
kind: Secret
metadata:
  name: private-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: git@bitbucket.org:your_account/your_repo.git
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----  
```

Apply the configuration and you should see your repository connected in ArgoCD UI!

ArgoCD related documentation: https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#ssh-repositories

