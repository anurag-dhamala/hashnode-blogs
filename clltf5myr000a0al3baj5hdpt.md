---
title: "SSH forwarding with Docker"
seoTitle: "SSH forwarding with Docker"
seoDescription: "How you can easily get access to private repository using Docker BuildKit? Read to find out how you can perform SSH forwarding with Docker."
datePublished: Sun Aug 27 2023 12:20:51 GMT+0000 (Coordinated Universal Time)
cuid: clltf5myr000a0al3baj5hdpt
slug: ssh-forwarding-with-docker
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/HSACbYjZsqQ/upload/774dc7b2b88eadedb5f44dafb979cb21.jpeg
tags: docker, web-development, devops, build, ssh

---

What is SSH forwarding with Docker? Why do we need it in the first place? What problem does it solve anyway? These are some of the questions that may immediately arise after reading the title. Bear with me, this article will address all of the questions along with its implementation.

### Scenario

Imagine a scenario: you are building a library that is to be used within an organization. As passionate as you are, you built it within a matter of days and now it's time to share your library with your fellow engineers. Since you cannot expose the library to the public, you plan to publish the library in a private repository that uses an SSH connection. Your teammates are happily using it as they too have access to that private repo. But, when it's time to build the docker image of the project that's using your library, the build process keeps failing. You are scratching your head on how to provide the docker image access to that private repo without exposing your private keys.

### Possible options

One possible option is to copy the private keys to your docker image, use it and remove it after the use. However, the metadata of the image will still hold that file containing the key.

Another option is to use the multistage build, copy the key and perform steps requiring the key in one stage and remaining in another stage. But, you need to be extra careful in this process and it can be quite complex depending upon your Dockerfile steps. Also, the private SSH keys are never meant to be copied like that in the first place.

So, there was no good approach until Docker introduced **the BuildKit builder** for the docker build process. Along with BuildKit, comes **SSH forwarding**.

### BuildKit

Buildkit is the builder backend for the docker build process. When you run the docker build command, docker uses the builder to build the docker image. Docker introduced Buildkit in docker version 18.09 and made it default since version 23.0 and onwards by replacing the legacy builder. It comes with tons of new features such as parallel execution, secret storage by mounting our secret files with --secret, improvements in logging and caching and much more. And, one of them is SSH forwarding. We will cover other features such as **secret storage (which can also be used in our scenario by the way)** in some other articles. Let's focus on SSH forwarding for now.

### SSH forwarding

With the BuildKit backend in the picture, we can forward our existing SSH agent connection. Note that, we are not copying our actual key during the build or anything like that. We are just informing BuildKit that, "Hey! SSH agent connection is forwarded". When the BuildKit encounters the RUN command that needs to access the remote server or repository through SSH, it simply requests the client to sign the request. Then, the SSH agent in the client signs the request. The private SSH key always stays in the client machine.

Let's move to the practical bits. If you are using the docker version 23.0 and above, you are using BuildKit by default. If this is not the case, you can enable the BuildKit during the docker build.

```bash
DOCKER_BUILDKIT=1 docker build .
```

Or you can export the environment variable using the "export" keyword if you do not want to mention DOCKER\_BUILDKIT every time.

We can use the **\--ssh flag** with the docker build command to forward our ssh-agent connection. You can also specify the path to your id\_rsa file if you have loaded it elsewhere. If it is in the default location i.e. **~/.ssh/id\_rsa**, you can just mention default. Let's assume that it is saved into the default ssh-agent.

```bash
DOCKER_BUILDKIT=1 docker build --ssh default -t image:tag .
```

That was it about the docker build command. Let's come to the Dockerfile.

When you are writing the Dockerfile, you should know which RUN command is going to need access to the SSH. For example: Say, you published your package in the private GitHub repo instead of the npm registry. *( I'm taking the example of a javascript library since it is the easiest).* The project that's using your library installed it through npm/yarn/pnpm. Locally, npm install command has been working fine since the project and the library both are in the same organization account and the fellow engineers have access to that library repository. The project's package.json *(a project that is using your library, not your library's package.json)* should look something like this:

```json
{
 name: "project-using-private-library",
 ......,
 dependencies: {
    "private-library": "git@github.com:<account>/<library-repo>.git"
 }
}
```

So, in this case, when you run **"npm install"** in one of the steps in Dockerfile, you will be requiring the SSH connection during that process (*Another example can be if you are writing RUN git clone &lt;your\_private\_repo&gt; in Dockerfile, then you will need SSH connection in this case too)*. Thus, we can let the builder know that the command needs access to the remote repo using SSH by using **\--mount=type=ssh** along with the run command.

```dockerfile
COPY package.json package.json
RUN --mount=type=ssh npm install
RUN npm run build
```

When the BuildKit encounters the RUN command with this flag, it will open the socket with read-only access to connect with SSH agent connection. The SSH agent in the client will then sign the request for our docker image. This socket will be mounted only when this step is running. Once, the command with the --mount=type=ssh flag is completed, this socket is closed by BuildKit and the commands below or above won't have any knowledge regarding this SSH agent connection. It won't leak to the other steps.

Hold on, it seems like we have done everything. But, aren't we missing something?

Remember when you clone something from GitHub or Bitbucket or whatever, a prompt appears asking that "the authenticity of the host couldn't be established. Do you want to continue connecting? \[Yes/No\]: "? There is no way of providing input to the prompt while building our image using the docker build command. So, we need to add the GitHub (or another remote server) to the **known hosts** before running the command with the SSH mount flag. How can we do that?

Well, we can install openssh and use the **ssh-keyscan** to download the public key for that host and add it to our **known\_hosts** file inside our docker container.

```dockerfile
FROM node:18-alpine as builder
WORKDIR /home/app
COPY package.json package.json

# apk add is a command to install package in alpine OS.
# you can use apt, dnf, pacman etc depending upon 
#your OS in docker image.
RUN apk add --no-cache openssh-client
RUN mkdir -p -m 0600 ~/.ssh
RUN ssh-keyscan github.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh npm install
RUN npm run build

#------- another stage -----------------
```

That's it. We installed open-ssh in our docker and created ".ssh" directory. 0600 is a chmod permission. Then we added a public key for the GitHub to the known\_hosts. Finally, we installed using the npm install command. You can implement a multi-stage build if you like *(reduces the final image size and also removes the actual code from the final image, so why not?)*.

By any chance, if you have your SSH key in some other location, I mentioned that you can specify the path. Here's the command to those guys:

```bash
docker build --ssh github=path/to/id_rsa -t image:tag .
```

Here, in **\--ssh github=path/to/id\_rsa**, github is just the id\*(can be anything)\* that we have to mention in the run command as shown below:

```dockerfile
RUN --mount=type=ssh,id=github npm install
#RUN --mount=type=ssh,id=github git clone git@github.com:<account>/<repo>.git
```

There you go! You can easily forward the ssh-agent connection using BuildKit while being assured that your private key is not exposed in any way in the docker image. If you read the whole article and found something useful, share it with your fellow engineers.