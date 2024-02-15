# Lab: container

## Dockerfile

- Dockerfile is a text file that contains instructions for building a Docker image
- Always specify a version in the base image
- `from` specifies the base image
  - **Base image does not contain the whole OS**
  - Alpine image for example mostly contains the fs of the OS

## `docker build`

```bash
docker build -t k8s-101 .
```

- builds a new image from the Dockerfile in the current directory
- `-t` tag the image with a name

## `docker run`

```bash
docker run -d -p 8080:80 --name k8s-101 k8s-101
```

- `-d` detached mode (run in background)
- `-p 8080:80` port mapping (host:container)
- `--name` name the container (instead of random hash)
- `k8s-101` image name

## `docker logs k8s-101`

- show logs of a container up to now
- `-f` follow the logs (live output)

## `docker exec k8s-101 nginx -v`

- execute a command in a running container
- `nginx -v` version of nginx
- `-it` interactive mode
  - can be used to enter a shell in the container
  - `docker exec -it k8s-101 sh`

## Multistage builds

- `FROM scratch` is a base image that only provides the functionality to run a binary
  - Google also provides similar images (distroless) for different languages (i. e. Python, Java, Node.js, etc.)

```dockerfile
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

- We can use multiple `FROM` statements in a Dockerfile i. e. multiple base images
  - The first image compiles the binary with the golang base image
  - The second image only contains the binary and runs it
- This is useful to keep the final image small and secure
  - Faster loading
  - Virtually no attack surface
- The last `FROM` defines the base image for the final image
