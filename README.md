1. git clone this repository:
`git clone https://github.com/lixuanbin/compile-openjdk-in-docker.git`
2. get in a specific directory, for example:
`cd ubuntu1404_openjdk8`
3. build the Docker image:
`docker build -t ubuntu1404_openjdk8:version3 .`
4. make local directory and run the image:
`mkdir -p ~/docker_share/ubuntu1404_openjdk8 && docker run -v ~/docker_share/ubuntu1404_openjdk8:/app -ti --entrypoint /bin/sh ubuntu1404_openjdk8:version3`
5. [inside container] copy the source and binaries to the mounted volume to persist your work:
`cp -r /opt/openjdk /app/ && cd /app/openjdk/openjdk8`
6. [inside container] alter the source codes and re-compile it, or you can use gdb to debug the source codes, have fun :)
