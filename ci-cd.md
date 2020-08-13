# Tooling

`podman` - manage pods / container images [run, stop, start, ps, attach, exec]
- Podman is used as a replacement for Docker

`buildah` - [build / push / signing] container images (SPECIALIZES IN BUILDS)
- Buildah has powerful build tools
- No Daemon
- Doesn’t include build tools within the image itself
    - Reduces the size of images you build
    - Image more secure
      - does not have software used to build the container 
        [gcc, make, and yum] within the resulting image.


# Buildah
An easy way to think of it is the `buildah run` command emulates the `RUN` command in a Dockerfile while the `podman run` command emulates the `docker run` command in functionality. 

```
$ cat > lighttpd.sh <<"EOF"
#!/usr/bin/env bash -x

ctr1=$(buildah from "${1:-fedora}")

## Get all updates and install our minimal httpd server
buildah run "$ctr1" -- dnf update -y
buildah run "$ctr1" -- dnf install -y lighttpd

## Include some buildtime annotations
buildah config --annotation "com.example.build.host=$(uname -n)" "$ctr1"

## Run our server and expose the port
buildah config --cmd "/usr/sbin/lighttpd -D -f /etc/lighttpd/lighttpd.conf" "$ctr1"
buildah config --port 80 "$ctr1"

## Commit this container to an image name
buildah commit "$ctr1" "${2:-$USER/lighttpd}"
EOF

$ chmod +x lighttpd.sh
$ sudo ./lighttpd.sh
```

# Buildah Signed Containers

-   **Creating Image Signatures**: Sign images with a private key and sharing a public key generated from it, others can use the public key to authenticate the images you share. The signatures needed to validate the images can be made available from an accessible location (like a Web server) in what is referred to as a "signature store" or made available from a directory on the local filesystem. The actual signature can be created from an image stored in a registry or at the time the image is pushed to a container registry.

-   **Verifying Signed Images**: You can check a signed image when it is pulled from a registry. This includes verifying Red Hat’s signed images.

-   **Trusting Images**: Besides determining that an image is valid, you can also set policies that say which valid images you trust to use on your system, as well as which registries you trust to use without validation.

# Trusted Software Supply Chain

![](https://raw.githubusercontent.com/jharmison-redhat/openshift-devsecops-labguides/develop/tekton/workshop/content/images/trusted_software_supply_chain.png)

# Image Registry

`Quay`

`OpenShift Image Registry`


# CI / CD Tooling 

`Jenkins`

`Tekton`


# OpenShift Pipeline Build

![](https://raw.githubusercontent.com/jharmison-redhat/openshift-devsecops-labguides/develop/tekton/workshop/content/images/openshift-pipeline.png)

**Overview**
-   Clone the git repository and make it available to the rest of the pipeline tasks.
-   Compile and packages the code using Maven
-   Execute the JUnit Test that exist in the same source tree.
-   Analyze the source code is analyzed for vulnerabilities, bugs, and bad patterns using SonarQube
-   Package the application as a WAR file, then pushes the WAR artifact to the Nexus Repository manager
-   Create a container image based the JBoss EAP runtime image and the content of the WAR artifact, then tag the newly created container image with the git SHA of the revision that was built
-   Deploy the newly created container image into the %username%-dev project
