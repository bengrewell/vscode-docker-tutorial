# Visual Studio Code with Docker

This walk through is designed to show how to use `Visual Studio Code` to develop and deploy applications
inside of docker containers on both local and remote machines. To follow along you will need to have
the latest version of `Visual Studio Code` and `Docker` on your local machine and if you wish to try the
remote docker deployment and debug techniques you will also need to have a remote system with `docker` 
installed and `ssh authentication` setup.

Below you will find some quick instructions for installing `Visual Studio Code` on your local system and
`Docker` on your local and remote systems. I'll also include instructions for setting up `ssh authentication`
on your remote system.

## Setting up prerequisites

#### Installing Visual Studio Code

Select one of the methods below based on your preferences and operating system.

Snap
```shell script
sudo snap install --classic code
```

**OR**

Debian / Ubuntu
```shell script
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
sudo apt install apt-transport-https
sudo apt update
sudo apt install code
```

**OR**

RHEL / Fedora / CentOS
```shell script
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf check-update
sudo dnf install code
```


#### Installing Docker

Select one of the methods below based on your preferences and operating system.

Snap

```shell script
sudo snap install docker
```

**OR**

Debian / Ubuntu

```shell script
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

**OR**

RHEL / Fedora / CentOS

```shell script
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
```

#### Setting up SSH Authentication

The steps below assume you already have a ssh key setup on your user account and just wish to authorize it
on a remote system. If you do not already have an ssh key then run `ssh-keygen` and follow the prompts.

```shell script
ssh-copy-id user@remote-host
```

#### Install VS Code Plugins

The final thing you will need to do is install a few plugins into `Visual Studio Code` to enable working with containers
to do this you need to complete the following steps

    1. Open Visual Studio Code
    2. Click on the extensions button on the left ribbon (it looks like 4 boxes with the top right detacted)
    3. Search for the `Docker` plugin
    4. Click on it and click on the `install` button
    5. Search for the `Remote-Containers` plugin
    6. Click on it and click on the `install` button

### Example application

If you already have an application you can skip forward, this section is just to help give some example code to
work with while you learn to use local and remote containers below. 

1. Open a terminal (you can do this in VS Code) and create a folder for your project
2. Click on "Open Folder" on the Getting Started window
3. Navigate to the folder you created for the project
3. In the explorer window under the name of your folder right click and select new file 
4. Name the file `test.py` and enter the following contents into the file then save it

    ```python
    import random
    
    x = random.randrange(100)
    print(x)
    ```

5. Click on the play button in the top right and make sure the code runs without an issue. It should print a value
between 0 and 100. If you run it again you should get a different value.

If this all worked then you should be ready to work with containers. What you just tested was running with a local
interpreter. In the next section under working with local docker containers we will run the same code inside a docker
container.

## Running code inside of fresh docker containers

#### Check Docker Plugin

Before we start deploying code into containers you will want to check and make sure that the plugin is working as
expected. To do this click on the Docker icon on the left ribbon. You should see "Containers", "Images", "Registries" ...
and other sections. If you expand them you should see a list of containers (if you have any) and images. If you have not
already now would be a good time to ensure docker is working as expected. You can do that by opening a terminal and
running the `hello world` container by entering the following command. 

```shell script
docker run hello-world
```

Running that command should produce the following output

```shell script
 ~  docker run hello-world                                                                                                                                             ✔ 
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete 
Digest: sha256:9f6ad537c5132bcce57f7a0a20e317228d382c3cd61edae14650eec68b2b345c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

In addition if you look back at `Visual Studio Code` under containers you should see your `hello-world` container and 
under images you should see `hellow-world` also. If that all works as expected then you should be ready to proceed. If
not then you should debug your `Docker` install which is outside of the scope of this document.

### Deploy application to Docker container

The first method we will cover will generate your `Dockerfile` and build an image which will then be deployed to your local
`Docker` instance and ran. 

1. Click `Ctrl` + `Shift` + `p` 
2. Type `Docker` at the prompt
3. Select `Docker: Add Docker Files to Workspace...`
4. Select `Python: General`
5. Select `test.py` or whatever you named your file
6. Select `Yes` when asked "Include Optional Docker Compose Files"

You should now have the following files in your project.

    .dockerignore
    docker-compose.yaml
    Dockerfile
    requirements.txt
    
Open the Dockerfile if it didn't automatically open for you. If you are not familiar with this format see below for a
quick intro.

```shell script
# Select the python 3.8 image to use for the containers base image
FROM python:3.8-slim-buster

# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1

# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1

# Install pip requirements
COPY requirements.txt .
RUN python -m pip install -r requirements.txt

# Set our working directory to /app
WORKDIR /app
# Copy our files into /app
COPY . /app

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-python-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

# During debugging, this entry point will be overridden. For more information, please refer to https://aka.ms/vscode-docker-python-debug
CMD ["python", "test.py"]
```

If you click on the `Run & Debug` icon in the left ribbon which looks like a play button with a bug on the bottom left
you should see `RUN...` and a play button with the words `Docker:Python` next to it. If you clikc on this it will build
your image with your application in it and then run it in a container. Once it is done building it should show you the
output of the application in the debug container.

### Run your code inside an existing Docker container

First we need to create an existing container to run our code in. To do this run the following instructions. Unlike our
last test this container will use a `Ubuntu` image and will have a entrypoint that doesn't exit so that it stays running. 
If you have your own container already you can use that. 

```shell script
docker run --name running-test -d ubuntu sleep infinity
```

The command should run and then return you to a prompt. If you look inside `Visual Studio Code` on the `Docker` tab you
should now see a running container called `ubuntu` with the name `running-test`.  

If you right click on that container and select `Attach Visual Studio Code` a second window should open up and should say
`Container ubuntu (running-test)` in the bottom left of the window. 

Next you would typically want to click on `Terminal` and then `New Terminal` which will open a terminal inside the
container. From here you could clone your repository. Since we are just doing a demo here we will simply create our
`test.py` file again. Click on `New File` on the `Getting Started` window. Click `Select Langage` and search for and
select `Python` then paste our code in again.

```python
import random

x = random.randrange(100)
print(x)
```  

You may be prompted to install the `Python Extensions` which you should do, this will install the extensions inside of 
the container. After it's done you may need to reload the window. 

Next install `Python` inside the container. *If apt update fails run the following* `echo nameserver 8.8.8.8 > /etc/resolv.conf` 

```shell script
apt update
apt install python3 python3-pip
```

Once this is done click on the play button in the top right and you should see your code run inside the container and
output the results as if you were running it on your local machine.

You can debug your code running in a container also but setting a breakpoint like you normally would. For example if you 
set a breakpoint on line 3 where you set the value of `x` and then go to `Run` and select `Start Debuggin` or simply
press `F5` and select `Python` when you're prompted. 

You code should run an break on line 3. If you press `F11` you should single step and you should now see the value of `x`
in the `Variables` pane to the left.

## Working with remote docker containers

To work inside an existing remote docker container you simply need to point to the remote docker client. To do that you 
do the following.

1. Click `File` -> `Preferences` -> `Settings`
2. Search for `DOCKER_HOST`
3. Set to `ssh://<user>@<address>`

You should setup SSH key based authentication for this so that you don't need to use user/password. To do this follow
these steps

1. If you don't have a ssh keypair generate one. If you do skip this
    ```shell script
    ssh-keygen
    <select the defaults>
    ```

2. Copy the ssh key to the remote system
    ```shell script
    ssh-copy-id <user>@<address>
    ```

3. Test
    ```shell script
    ssh <user>@<address> whoami
    ```
   
This should run the command without prompting for a password and return your username. If this works then
you should be all setup for using a remote docker container. 

If you return to the `Visual Studio Code` window on the docker tab you should now see the containers and images
on your remote system instead of your local system. If you right click on your running container and select `Attach
 Visual Studio Code` you will get a new window attached to that remote container. 

*You may get an error about Visual Studio Code not being able to connect to the container.* 
*If you do then just click `Chose Container` and select your container again.* 
 
You might need to click on the extensions and install the Python extension into the container. Once this is done you
should be able to develop against the remote container as if you were using it locally.