# Build version 1.0 of IDE
```sh
version=v4.0
docker build --platform=linux/amd64 -t cb-ide:${version} -f browser.Dockerfile .
docker image ls | grep cb-ide
docker login
docker tag cb-ide:${version} terrydhariwal/cb-ide:${version}
docker push terrydhariwal/cb-ide:${version}
```

# Start version 1.0 of IDE plus Bandersnatch (pypi mirror)
```sh
# from project root start the ide:
docker run -p=80:3000 --rm -d terrydhariwal/cb-ide:${version}
ssh -v -p 80 localhost
```


```
IMAGE_TO_SEARCH=terrydhariwal/cb-ide:${version}
CONTAINER_NAME_TO_CONNECT_WITH=$(docker ps --filter ancestor=${IMAGE_TO_SEARCH} --format "{{.Names}}")
docker exec -it ${CONTAINER_NAME_TO_CONNECT_WITH} /bin/bash
```

# Docker-Compose with IDE, Nexus and Bandersnatch
```
cd docker-compose-ultimate
docker compose up -d
docker compose ps

# create maven directory in theia-ide and configure permissions for user 101
sudo su - 
volume_name=docker-compose-ultimate_theia-home
mount_point=$(docker volume inspect $volume_name | jq -r '.[0].Mountpoint')
echo "The mount point of the volume is: $mount_point"
sudo mkdir -p ${mount_point}/.m2
sudo ls ${mount_point}/

# create ~/.m2/settings.xml file referencing nexus 
export NEXUS_USERNAME=admin
export NEXUS_PASSWORD=password
cat <<EOF > ${mount_point}/.m2/settings.xml
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://${HOSTNAME}:443/repository/maven-central/</url>
    </mirror>
  </mirrors>
  <servers>
    <server>
      <id>nexus</id>
      <username>${NEXUS_USERNAME}</username>
      <password>${NEXUS_PASSWORD}</password>
    </server>
  </servers>
</settings>
EOF

cat ${mount_point}/.m2/settings.xml
# fix permissions so that container has access to generated files
chown -R 101:101 ${mount_point}/.m2
ls -l ${mount_point}/.m2


# get nexus admin password
volume_name="docker-compose-ultimate_nexus-data"
mount_point=$(docker volume inspect $volume_name | jq -r '.[0].Mountpoint')
echo "The mount point of the volume is: $mount_point"
sudo ls ${mount_point}/
sudo cat ${mount_point}/admin.password; echo

# exit root
exit 

# restart containers to take effect of new files? not sure i need to do this?
docker compose restart
```

```shell
docker compose ps
NAME                                           IMAGE                       COMMAND                  SERVICE              CREATED         STATUS         PORTS
docker-compose-ultimate-cb-ide-1               terrydhariwal/cb-ide:v1.0   "node /home/theia/ap…"   cb-ide               5 minutes ago   Up 5 minutes   0.0.0.0:80->3000/tcp
docker-compose-ultimate-nexus-1                sonatype/nexus3             "/opt/sonatype/nexus…"   nexus                5 minutes ago   Up 5 minutes   0.0.0.0:8081->8081/tcp

docker-compose-ultimate-cb-ide-1 = IDE
github/gitlab/bitbucket = code repo
docker-compose-ultimate-nexus-1  = java/python package repo
dockerhub = docker repo
mvm/gradle = package manager = connect with java package repo and downloads packages and dependecies for you
```
