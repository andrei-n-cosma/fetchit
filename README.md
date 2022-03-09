# Harpoon
The purpose of Harpoon is to allow for GitOps management of podman managed containers.

This project is currently under development.

## Requirements:
To compile harpoon outside of the Dockerfile the following must be installed on the system:

On Fedora:
- gpgme-devel
- libbtrfs
- btrfs-progs-devel
- device-mapper-devel

##  Running
Harpoon requires the podman socket to be running on the host. The socket can be enabled for a specific user or for root.

To enable the socket for $USER:
```
systemctl --user enable podman.socket --now
```

To enable the socker for root:
```
systemctl enable podman.socket
```

Define the parameters in your `config.yaml` to relate to your git repository.
```
targets:
- name: harpoon
  url: http://github.com/redhat-et/harpoon
  branch: main
  systemd:
    targetPath: examples/systemd/httpd.service
    schedule: "*/1 * * * *" 
  raw:
    targetPath: examples/raw
    schedule: "*/1 * * * *"
```
Verify running characters before deploying harpoon.
```
podman ps

CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```


Launch the harpoon container using a podman volume.

NOTE: If a podman volume is not the preferred storage solution a directory can be used as well. An example would be `-v ~/harpoon-volume:/opt` instead of `-v harpoon-volume:/opt`.

```
podman run -d --name harpoon -v harpoon-volume:/opt -v ./config.json:/opt/config.json -v /run/user/$(id -u)/podman//podman.sock:/run/podman/podman.sock quay.io/harpoon/harpoon:latest
```

The container will be started and will run in the background. To view the logs:

```
podman logs -f harpoon

git clone http://github.com/redhat-et/harpoon main --recursive
Creating podman container from ./harpoon/examples/raw/example.json
Trying to pull docker.io/mmumshad/simple-webapp-color:latest...
Getting image source signatures
Copying blob sha256:b023afffd10b07f646968c0f1405ac7b611feca6da6fbc2bb8c55f2492bdde07
Copying blob sha256:d4eee24d4dacb41c21411e0477a741655303cdc48b18a948632c31f0f3a70bb8
Copying blob sha256:1607093a898cc241de8712e4361dcd907898fff35b945adca42db3963f3827b3
Copying blob sha256:b59856e9f0abefedc34fcefc3f57c4955cc384785663745532ddc31a89641c83
Copying blob sha256:55cbf04beb7001d222c71bfdeae780bda19d5cb37b8dbd65ff0d3e6a0b9b74e6
Copying blob sha256:13e2e806d7c88f357958d798c097b4fc0cd6e3aea888ad7e584fba5a0e7d3ec9
Copying blob sha256:e90bc178f0458c231d8e355756f9f0f51e22a4e6c5ff8c9c6cb8e48d2c158000
Copying blob sha256:bd415728f75acd3ee7699f4bb31dfa8c39a935d5a6acea4b580568cd730100a9
Copying blob sha256:06d08c7638af6fc0c05f9c7e5ec43ae7b24ca72bbfaba4d065578358ed38ab15
Copying blob sha256:98b4690dc1c724ec64b18475f1be8d37e10c058788da16aa2e4ca7260c1aac68
Copying blob sha256:3a4e7915e2111a1546b662863d4192f98283e53a66ac34296d90823563d12040
Copying blob sha256:b2567acc3f180ce1113a668c5950e8123b493c5b85e8e51651310ee21799c67d
Copying blob sha256:9a8ea045c9261c180a34df19cfc9bb3c3f28f29b279bf964ee801536e8244f2f
Copying config sha256:96bb69733441c4a81ec77348208198aba7a5a78f4dc22429e7a56b25f63d2b73
Writing manifest to image destination
Storing signatures
A container named colors already exists. Removing the container before redeploy.
Container created.
time="2022-02-15T18:04:14Z" level=info msg="Going to start container \"53d86851aad9fc362cb61493c495ec262217c1759e061724dc1f974c35d93d5b\""
Container started....Requeuing
```

Verify the sample application is running
```
podman ps

CONTAINER ID  IMAGE                                          COMMAND               CREATED         STATUS             PORTS                   NAMES
53d86851aad9  docker.io/mmumshad/simple-webapp-color:latest  python ./app.py       53 minutes ago  Up 53 minutes ago  0.0.0.0:8080->8080/tcp  colors
```
