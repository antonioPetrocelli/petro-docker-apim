# Dockerfile for WSO2 API Manager #
[Source of this doc](https://github.com/wso2/docker-apim/blob/4.2.x/dockerfiles/centos/apim/README.md)

[GitHub Source files](https://codeload.github.com/wso2/docker-apim/zip/refs/tags/v4.2.0.1)

This section defines the step-by-step instructions to build an [CentOS](https://hub.docker.com/_/centos/) Linux based Docker image for WSO2 API Manager 4.2.0.

## Prerequisites

* [Docker](https://www.docker.com/get-docker) v17.09.0 or above
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) client


## How to build an image and run

#### 1. Checkout this repository into your local machine using the following Git client command.

```
git clone https://github.com/antonioPetrocelli/petro-docker-apim.git
```

> The local copy of the `C:\Users\anton\OneDrive\Documenti\GitHub\petro-docker-apim\dockerfiles\centos\apim` directory will be referred to as `AM_DOCKERFILE_HOME` from this point onwards.

#### 2. Build the Docker image.

- Navigate to `<AM_DOCKERFILE_HOME>` directory. <br>
- Change <APIM_DIST_URL> in Dockerfile to the location of the product pack. (http://some_domain_or_ip/wso2/wso2am-4.2.0.zip)
- Execute `docker build` command as shown below.

```
docker build -t wso2am:4.2.0-centos .
```

> By default, the Docker image will prepackage the General Availability (GA) release version of the relevant WSO2 product.

#### 3. Push the Docker image to your DockerHub Repository.

* Get images list
```
docker image ls
```

|REPOSITORY                    |TAG            |IMAGE ID       |CREATED         |SIZE   |
|------------------------------|---------------|---------------|----------------|-------|
|wso2am                        |4.2.0-centos   |ab3ee76c2379   |9 minutes ago   |1.28GB |
|gcr.io/k8s-minikube/kicbase   |v0.0.43        |619d67e74933   |2 weeks ago     |1.26GB |

* Push command sample
```
docker tag local-image:tagname petroanvasys/imagename:tagname
docker push petroanvasys/imagename:tagname
Make sure to replace tagname with your desired image repository tag.
```

* Push the image
<!---cGV0cm9hbnZhc3lzOlRVSjdBaHdxVDR4Wjku-->
```
docker login -u "docker_user" -p "docker_password" docker.io
docker tag wso2am:4.2.0-centos petroanvasys/wso2am:4.2.0-centos
docker push petroanvasys/wso2am:4.2.0-centos
```

#### 4. Running the Docker image.

```
docker run -it -p 9443:9443 -p 8243:8243 wso2am:4.2.0-centos
```

> Here, only port 9443 (HTTPS servlet transport) and port 8243 (Passthrough or NIO HTTPS transport) have been mapped to Docker host ports.
You may map other container service ports, which have been exposed to Docker host ports, as desired.
If you want run the docker image with all ports enabled, run this command

```
docker run -it -p 9443:9443 -p 8243:8243 -p 9763:9763 -p 9999:9999 -p 11111:11111 -p 8280:8280 -p 5672:5672 -p 9711:9711 -p 9611:9611 -p 9099:9099 wso2am:4.2.0-centos
```

#### 5. Accessing management console.

- To access the management console, use the docker host IP and port 9443.
    + `https://<DOCKER_HOST>:9443/carbon`
    
> In here, <DOCKER_HOST> refers to hostname or IP of the host machine on top of which containers are spawned.

| HTTP                                 | URL                                    |
|--------------------------------------|----------------------------------------|
|Mgt Console URL:                      |http://localhost:9763/carbon/           |
|API Admin Portal Dashboard:           |http://localhost:9763/admin/dashboard   |
|API Developer Portal Default Context: |http://localhost:9763/devportal         |
|API Publisher Default Context:        |http://localhost:9763/publisher         |
|API Manager Gateway:                  |http://localhost:8280/services/Version  |

| HTTPS                                | URL                                    |
|--------------------------------------|----------------------------------------|
|Mgt Console URL:                      |https://localhost:9443/carbon/          |
|API Admin Portal Dashboard:           |https://localhost:9443/admin/dashboard  |
|API Developer Portal Default Context: |https://localhost:9443/devportal        |
|API Publisher Default Context:        |https://localhost:9443/publisher        |
|API Manager Gateway:                  |https://localhost:8243/services/Version |


## How to update configurations

Configurations would lie on the Docker host machine and they can be volume mounted to the container. <br>
As an example, steps required to change the port offset using `deployment.toml` is as follows:

#### 1. Stop the API Manager container if it's already running.

In WSO2 API Manager version 4.2.0 product distribution, `deployment.toml` configuration file <br>
can be found at `<DISTRIBUTION_HOME>/repository/conf`. Copy the file to some suitable location of the host machine, <br>
referred to as `<SOURCE_CONFIGS>/deployment.toml` and change the offset value (`[server]->offset`) to 1.

#### 2. Grant read permission to `other` users for `<SOURCE_CONFIGS>/deployment.toml`.

```
chmod o+r <SOURCE_CONFIGS>/deployment.toml
```

#### 3. Run the image by mounting the file to container as follows:

```
docker run -it \
-p 9444:9444 \
-p 8244:8244 \
--volume <SOURCE_CONFIGS>/deployment.toml:<TARGET_CONFIGS>/deployment.toml \
wso2am:4.2.0-centos
```

> In here, <TARGET_CONFIGS> refers to /home/wso2carbon/wso2am-4.2.0/repository/conf folder of the container.

## Running official Ubuntu wso2am images
It is possible to use official wso2am images without building them from the scratch.

- To run on amd64 or Apple Silicon (arm64)
```
docker run -it -p 9443:9443 -p 8243:8243 wso2/wso2am:4.2.0-centos
```
> This official image is built for amd64 thus it will not run on Apple silicon natively. But it will run on emulated docker on Rosetta.

## How to build a Docker image with multi architecture support

The above wso2am:4.2.0 image will only be supported for the CPU architecture of your current machine. Docker buildx plugin can be used to build wso2am:4.2.0 image to support any CPU architecture.

#### 1. Install [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/)

#### 2. Install [QEMU Emulators](https://github.com/tonistiigi/binfmt)
```
docker run -it --rm --privileged tonistiigi/binfmt --install all
```

#### 3. Create, switch and inspect a new builder
```
docker buildx create --name wso2ambuilder
```
```
docker buildx use wso2ambuilder
```
```
docker buildx inspect --bootstrap
```
#### 4. Build and push 

```
docker buildx build --platform linux/amd64,linux/arm64 -t <DOCKER_USERNAME>/wso2am:4.2.0-centos-multiarch --push .
```

> - Here <DOCKER_USERNAME> is a valid Docker or Dockerhub username.
> - Use command "docker login" to authenticate first if it fails to push.
> - You can specify any number of platforms to support --platform flag
> - Use command "docker buildx ls" to see list of existing builders and supported platforms.
> - Please note we have only tested this for linux/amd64 and linux/arm64 platforms only

#### 5. Run
```
docker run -it -p 9443:9443 -p 8243:8243 <DOCKER_USERNAME>/wso2am:4.2.0-centos-multiarch
```
> Docker will pull the suitable image for the architecture and run

> **Note**
> If you are using Rancher to run the Docker image, you will not be able to use port 9443, which is already allocated by Rancher. As a workaround, you can follow the instructions given in [How to update configurations](#how-to-update-configurations) to run the APIM image in a different port.

## Docker command usage references

* [Docker build command reference](https://docs.docker.com/engine/reference/commandline/build/)
* [Docker run command reference](https://docs.docker.com/engine/reference/run/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [Docker multi architecture build reference](https://docs.docker.com/desktop/multi-arch/)
* [Docker buildx reference](https://docs.docker.com/buildx/working-with-buildx/)
