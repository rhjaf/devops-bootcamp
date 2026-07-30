## Local Docker registry setup
In this task, we will setup a Local Docker registry.
1. Create 3 directory for storing credntials
    ```bash
    mkdir -p ~/registry/{auth,data,certs}
    ```
2. Install `htpasswd` which is used to define credntial for authorization with http-based services
    ```bash
    sudo apt install apache2-utils -y
    ```
3. Create your customer user (here the username is dockeruser) and set its password
    ```bash
    htpasswd -Bc ~/registry/auth/htpasswd dockeruser
    ```
    the credintials are going to be stored in `registry/auth/htpasswd`.
3. in DockerHub, there is an image specific for creating a local docker registry which we are going to use it
    ```bash
    docker run -d --restart always --name local-docker-registry -p 5000:5000 \
    -v ~/registry/data:/var/lib/registry -v ~/registry/auth:/auth \
    -e REGISTRY_AUTH=htpasswd -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd registry
    ```
4. Now we need connect to our repo using our username and password define above
    ```bash
    docker login 192.168.33.131:5000
    ```
5. Finally we can pull/push docker images from/to our local docker registry
    ```bash
    docker pull alpine:latest
    docker tag alpine:latest 192.168.33.131:5000/alpine:latest
    docker push 192.168.33.131:5000/alpine:latest
    ```
    Remeber that you need to add `insecure-registries` to you `daemon.json` file if the connection to repo is not through HTTPs.
