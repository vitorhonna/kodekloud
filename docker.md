# Course: Docker Training Course for the Absolute Beginner

## Basic Commands

### Run a Container from an Image

* `docker run <image>`
* `docker run <image> <command>`
* `docker run -d <image>`
* `docker run -d --name <name> <image>`

### Run commands in a Container

* `docker exec -it <container> <command>`
  - Example: `docker exec -it my_container bash`

### Attach to a Running Container

* `docker attach <container>`

### Stop a Container

* `docker stop <container>`

### Remove a Container

* `docker rm <container>`

### List Containers

* `docker ps`
  - Lists running containers only.
* `docker ps -a`
  - Lists all containers

### List Images

* `docker images`

### Remove Images

* `docker rmi <image>`

### Pull an Image

* `docker pull <image>`
  - Only pulls the image, it does not run it.

## Docker Run

### Run a Container from an Image specifying a Tag

* `docker run <image>:<tag>`
  - The default tag is "latest"
  - Example: `docker run nginx:1.19` runs the nginx image with the tag 1.19.

### Run a Container interactively

* `docker run -it <image>`
  - Runs a container interactively with a terminal attached.
  - `-i`: interactive, redirects the STDIN for the terminal to the container
  - `-t`: terminal, attaches the terminal to the container to receive the STDOUT

### Run a Container and remove it after it stops

* `docker run --rm <image>`
  - Automatically removes the container when it stops.
  - Useful for one-off tasks or scripts.

### Port Mapping

* `docker run -p <host_port>:<container_port> <image>`
  - Maps a port on the host to a port on the container.
  - Example: `docker run -p 8080:80 nginx` maps port 8080 on the host to port 80 on the nginx container.

### Volume Mapping

* `docker run -v <host_path>:<container_path> <image>`
  - Maps a directory on the host to a directory in the container.
  - This is useful for persisting data or sharing data between the host and the container.
  - Example: `docker run -v /opt/data:/var/lib/mysql mysql` maps `/opt/data` on the host to `/var/lib/mysql` in the MySQL container.

### Inspect a Container

* `docker inspect <container>`
  - Displays detailed information about a container in JSON format.
  - Useful for debugging and understanding the configuration of a container.

### Container Logs

* `docker logs <container>`
  - Displays the logs of a container.
  - Useful for debugging and monitoring the output of a container.

## Docker Images

Before starting, list the steps it would take to manually deploy the application without Docker. Then, create a `Dockerfile` that automates those steps.

For example, for a simple Flask application, the manual steps might include:

1. VM running Ubuntu
2. Update apt repo
3. Install dependencies using apt
4. Install Python and dependencies using pip
5. Copy the source code the the `/opt` directory
6. Run the web server using the `flask` command

To create a Docker image, first create the `Dockerfile`:

```Dockerfile
FROM ubuntu

# Update apt repo
RUN apt-get update

# Install dependencies using apt
RUN apt-get install -y python3 python3-pip

# Install Python dependencies using pip
COPY requirements.txt /opt/
RUN pip3 install -r /opt/requirements.txt

# Copy the source code the the /opt directory
COPY . /opt/source-code

# Run the web server using the flask command
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Then, build the image using the `docker build` command:

```bash
docker build -f Dockerfile -t my_flask_app .
```
This command builds the Docker image (locally) and tags it as `my_flask_app`.

To make it available to others, you can push it to a Docker registry like Docker Hub:

```bash
docker tag my_flask_app myusername/my_flask_app:latest
docker push myusername/my_flask_app:latest
```

Finally, run a container from the image:

```bash
docker run -d -p 5000:5000 my_flask_app
```
This command runs the container in detached mode and maps port 5000 on the host to port 5000 in the container.

### Environment Variables

Export a variable in the host:

```bash
export FLASK_ENV=dev
```

Pass the variable to the container:

```bash
docker run -d -p 5000:5000 -e FLASK_ENV my_flask_app
```

 To find the environment variables in a running container, inspect it and check the ``Env`` array:

```bash
docker inspect <container_id> | grep Env -A 10
```

### Commands vs Entrypoint

#### CMD

`CMD` specifies the default command to run when the container starts. It can be overridden by providing a different command when running the container. It's usually placed at the end of the `Dockerfile` and defined as an array of strings.

* Example:
  ```Dockerfile
  CMD ["sleep", "5"]
  ```
  * If run with `docker run <image>`, it will execute `sleep 5`.
  * If run with `docker run <image> sleep 10`, it will execute `sleep 10` instead.

#### ENTRYPOINT

`ENTRYPOINT` specifies a command that will always run when the container starts. It cannot be overridden by providing a different command when running the container. It's usually placed at the end of the `Dockerfile` and defined as an array of strings.

* Example 1:
  ```Dockerfile
  ENTRYPOINT ["sleep", "5"]
  ```
  * If run with `docker run <image>`, it will execute `sleep 5`.
  * If run with `docker run <image> sleep 10`, it will still execute `sleep 5`.

* To override the `ENTRYPOINT`, use the `--entrypoint` flag when running the container:
  ```bash
  docker run --entrypoint echo <image> 10
  ```
  * It will execute `echo 10`.

* Example 2:
  ```Dockerfile
  ENTRYPOINT ["sleep"]
  ```
  * If run with `docker run <image>`, it will result in an error because no argument is provided to `sleep`.
  * If run with `docker run <image> 10`, it will execute `sleep 10`.

* Note: If you use `ENTRYPOINT`, it's a good practice to also use `CMD` to provide default arguments that can be overridden (see below).

#### Using both CMD and ENTRYPOINT

If both `CMD` and `ENTRYPOINT` are specified, the `CMD` will be passed as arguments to the `ENTRYPOINT`.

* Example:
  ```Dockerfile
  FROM ubuntu
  ENTRYPOINT ["sleep"]
  CMD ["5"]
  ```
   * If run with `docker run <image>`, it will execute `sleep 5`.
   * If run with `docker run <image> 10`, it will execute `sleep 10`.

