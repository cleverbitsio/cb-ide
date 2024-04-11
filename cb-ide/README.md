# Build version 1.0 of IDE
```sh
docker build --platform=linux/amd64 -t cb-ide -f browser.Dockerfile .
docker image ls | grep cb-ide
docker login
docker tag cb-ide:v1.0 terrydhariwal/cb-ide:v1.0
docker push terrydhariwal/cb-ide:v1.0
```

# Start version 1.0 of IDE plus Bandersnatch (pypi mirror)
```sh
# from project root start the ide:
docker run -p=80:3000 --rm -d terrydhariwal/cb-ide:v1.0
ssh -v -p 80 localhost

# Using docker compose I'll also spin up a local mirror of pypi
# https://github.com/pypa/bandersnatch/tree/main / https://github.com/pypa/bandersnatch/tree/main/src/bandersnatch_docker_compose
# run python mirror
cd cb-ide/bandersnatch/bandersnatch_docker_compose
docker compose up -d
docker ps | grep bander 
# 9f0da05acecb   bandersnatch_nginx              "/docker-entrypoint.…"   25 hours ago   Up 25 hours           0.0.0.0:40080->80/tcp    bandersnatch_docker_compose-bandersnatch_nginx-1
ssh -v -p 40080 localhost

# Using docker compose I'll also spin up a local mirror of Java
#...
```

connect to <http://localhost:80/>
connect to <http://localhost:40080/>

# Populate Python Mirror Quickstart:
`bandersnatch/bin/bandersnatch --help`
Run `bandersnatch mirror` - it will create an empty configuration file for you in /etc/bandersnatch.conf.
Review `/etc/bandersnatch.conf` and adapt to your needs.
Run `bandersnatch mirror` again. It will populate your mirror with the current status of all PyPI packages. Current mirror package size can be seen here: https://pypi.org/stats/
A `blocklist` or `allowlist` can be created to cut down your mirror size. You might want to Analyze PyPI downloads to determine which packages to add to your list.
Run `bandersnatch mirror` regularly to update your mirror with any intermediate changes.


```

#IMAGE_TO_SEARCH=terrydhariwal/cb-ide:v1.0
#IMAGE_TO_SEARCH=bandersnatch_nginx
IMAGE_TO_SEARCH=pypa/bandersnatch:latest
CONTAINER_NAME_TO_CONNECT_WITH=$(docker ps --filter ancestor=${IMAGE_TO_SEARCH} --format "{{.Names}}")
docker exec -it ${CONTAINER_NAME_TO_CONNECT_WITH} /bin/bash

bandersnatch --help

```

# Docker-Compose with IDE, Nexus and Bandersnatch
```
cd docker-compose-ultimate
docker compose up -d
docker compose ps

# get admin password
volume_name="docker-compose-ultimate_nexus-data"
mount_point=$(docker volume inspect $volume_name | jq -r '.[0].Mountpoint')
echo "The mount point of the volume is: $mount_point"
sudo cat ${mount_point}/admin.password; echo

~/.m2/settings.xml
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://localhost:8081/repository/maven-central-proxy/</url>
    </mirror>
  </mirrors>
</settings>

```

```shell
docker compose ps
NAME                                           IMAGE                       COMMAND                  SERVICE              CREATED         STATUS         PORTS
docker-compose-ultimate-bandersnatch-1         pypa/bandersnatch:latest    "bandersnatch --conf…"   bandersnatch         5 minutes ago   Up 5 minutes   
docker-compose-ultimate-bandersnatch_nginx-1   bandersnatch_nginx          "/docker-entrypoint.…"   bandersnatch_nginx   5 minutes ago   Up 5 minutes   0.0.0.0:40080->80/tcp
docker-compose-ultimate-cb-ide-1               terrydhariwal/cb-ide:v1.0   "node /home/theia/ap…"   cb-ide               5 minutes ago   Up 5 minutes   0.0.0.0:80->3000/tcp
docker-compose-ultimate-nexus-1                sonatype/nexus3             "/opt/sonatype/nexus…"   nexus                5 minutes ago   Up 5 minutes   0.0.0.0:8081->8081/tcp

docker-compose-ultimate-cb-ide-1 = IDE
github/gitlab/bitbucket = code repo
docker-compose-ultimate-nexus-1  = java package repo
docker-compose-ultimate-bandersnatch_nginx-1  = python package repo
dockerhub = docker repo
mvm/gradle = package manager = connect with java package repo and downloads packages and dependecies for you
```

# todo
# Building extended version of cb-ide based of image cb-ide:v1.0
```bash
cd cb-ide
# extended build to install additional tools etc
docker build -t cb-ide:v2.0 -f Dockerfile .
docker run -p=80:3000 --rm -d cb-ide:v2.0
```

