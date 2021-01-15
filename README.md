# Docker Credential Helper for AWS ECR via AWS SSO

This credential helper allows for seamless access to Elastic Container Registry repositories via SSO enabled AWS accounts.

It works by parsing the repository URL and searching your AWS CLI config file for the appropriate account information. It then performs
the required credential fetching dance with AWS, and returns the authentication information to docker.

# Setup
This helper requires you to have `bash`, `awk`, and `awscli` (version 2+) installed.

1) Place the file `docker-credential-aws-sso-ecr` somewhere on your `PATH`. I put it at `/usr/local/bin/docker-credential-aws-sso-ecr`.
2) Update your docker config (normally at `$HOME/.docker/config.json`) and tell it to use this new credential helper for your ECR repositories.
The URL key should match your repository, the value should match the file name of the helper, without the `docker-credential-` prefix
e.g.: 
```
<snip>
    "credHelpers": {
        "<ACCOUNT-ID>.dkr.ecr.<REGION>.amazonaws.com: "aws-sso-ecr"
    }
</snip>
```

# Use Docker
If your AWS CLI isn't currently authenticated to ECR, attempting to pull an image (or otherwise interact with the repository) may look like this:

```
➜ docker pull <ACCOUNT-ID>.dkr.ecr.<REGION>.amazonaws.com/<REPOSITORY>

The SSO session associated with this profile has expired or is otherwise invalid. To refresh this SSO session run aws sso login with the corresponding profile.
Error response from daemon: Head https:<ACCOUNT-ID>.dkr.ecr.<REGION>.amazonaws.com/<REPOSITORY>: no basic auth credentials
```

If you see this, you'll need to re-authenticate your CLI with AWS:

```
➜ aws sso login --profile <my-profile>
```

And then your docker interactions will work again:
```
➜ docker pull <ACCOUNT-ID>.dkr.ecr.<REGION>.amazonaws.com/<REPOSITORY>
Using default tag: latest
latest: Pulling from <REPOSITORY>
```


# Testing
I've tested this with an AWS CLI configured with two SSO-enabled profiles.

I have not tested this for any profiles that authenticate to one region and attempt to access AWS ECR repositories in another

I've tested this on MacOS 11.0.1 with the following docker client version:
```
➜ docker version
Client: Docker Engine - Community
 Cloud integration: 1.0.4
 Version:           20.10.0
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        7287ab3
 Built:             Tue Dec  8 18:55:43 2020
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.0
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       eeddea2
  Built:            Tue Dec  8 18:58:04 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
