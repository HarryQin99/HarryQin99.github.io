---
layout: post
title: 🗂 【Exploration】Golang Docker K8S
date: 2023-04-02 00:00
---

Nowaday, using docker with kubernetes as the deployment tools is more and more popular. This exploration document here will mainly focuse on **how** we use docker and kubernetes to deploy a simple service (fully local, yes even the cluster we going to use in kubernetes is in local as well 🥳)

<br />
# Agenda
- [Develop a simply service](#Develop a simply service)
- [Create image for this service](#Create image for this service)
- [Using docker image to deploy the service](#Using docker image to deploy the service)

---

&nbsp;
# Develop a simple service
### Prerequires
```
mkdir ping-service
cd ping-service
go mod init main
touch main.go
```
### Implementation
Ping service is a good example as a simple service. Requirement for ping service is it keeps sending get request to a url provided. If there is no error happens, then log the url. Otherwise, log the error it gets. Also we want to stop the pogram when it received unix signals like SIGTERM or SIGINT

<ins>golang code</ins>:
```go
func main() {

	// Get url from env
	url := os.Getenv("URL")

    // Use go channel to ping url
	go pingUrl(url)

    // Stop while 
	stop := make(chan os.Signal, 1)

	signal.Notify(stop, syscall.SIGINT, syscall.SIGTERM)

	<-stop

	log.Println("Shut down")
}

//  Function which to keep sending get request to url provided and with 10 seconds sleep before next request
func pingUrl(url string) {
	for {
		_, err := http.Get(url)
		if err != nil {
			log.Println(err.Error())
		}
		log.Println(url)
		time.Sleep(10 * time.Second)
	}
}
```

### Test to run:
```bash
URL=http://www.google.com go run main.go
```
![](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/Ping-Service.png)

---

&nbsp;
# Create image for this service
### Prerequires
```
touch dockerfile
```

### Implementation
After we have a runable service, the next setp is to create image for this service.

<ins>dockfile</ins>
```dockerfile
FROM golang:1.16-alpine # Specific a base image to use, all following command is running under this base image. Here we use golang image cause our project is written in golang

WORKDIR /app # Create a default directory, it it also the default destination for following command

COPY . /app # Copy the whole project to the image, this could be "COPY . ./" as well, cause we already set up /app as the default destination

RUN go build -o /ping-service # Compile the go program and the application binary name is "ping-service"

CMD [ "/ping-service" ] # Run ping-service which we compiled in previous step
```

<ins>build the image</ins>

Specific the image name and version in format of {image_name:image_version} after --tag (if only specific the image_name, then its tag will be latest as default). At the end, speficic the path of where dockerfile is. 
```command
docker build --tag ping-service .

docker images | grep "ping-service"
```
![](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/Ping-service-build-image.png)

<ins>test to run the image</ins>

After we created the image, can eaisly test if the image is doing what we want to do by running the image locally
```command
docker run --env URL=http://www.google.com ping-service
```
![](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/Ping-service-run-image.png)

---

&nbsp;
# Deploy the service using the created image in K8S

### Prerequires
<ins>using "kind" to setup your cluster locally</ins>

Follow the instruction [here](https://kind.sigs.k8s.io/docs/user/quick-start/) to setup cluster locally. 

Once your cluster setup, confirm by `kubectl cluster-info`

![](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/Ping-service-kind-cluster.png)

<ins>load image to cluster</ins>

Cause we didn't push our image to docker hub. To let the cluster able to pull the image we want while deploying the service, we need to load the image to the cluster first.
```dockerfile
# ping-service is the image you want to load into the cluster. Specific the name of cluster after --name
kind load docker-image ping-service  --name kind
```

### Implementation
```
touch ping-service.yml
```
<ins>configuration of the deployment</ins>
```yml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: ping-service-deployment

  labels:

    app: ping-service

spec:

  replicas: 1

  selector:

    matchLabels:

      app: ping-service

  template:

    metadata:

      labels:

        app: ping-service

    spec:

      containers:

      - name: app

        image: ping-service # specific the image 
        imagePullPolicy: IfNotPresent

        env:
          - name: URL # specific the env value needed
            value: http://www.google.com

```
<ins>apply deployment</ins>
```bash
kubectl apply -f ./ping-service.yml

kubectl get pods # check if the service is running as expected

kubectl logs {pod-name} -f # check the log
```
![](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/Ping-serevice-final.png)

After this, you get a existing ping-service running in your local which using K8S and based on the docker image you compiled. WoW

---
### Referenece
[Build your Go image](https://docs.docker.com/language/golang/build-images/)

[K8S deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

[Use kind to setup a cluster locally](https://kind.sigs.k8s.io/docs/user/quick-start/)