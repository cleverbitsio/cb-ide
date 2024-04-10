# Build version 1.0 of IDE
```sh
docker build -t cb-ide -f browser.Dockerfile .
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

# Building extended version of cb-ide based of image cb-ide:v1.0
```bash
cd cb-ide
# extended build to install additional tools etc
docker build -t cb-ide:v2.0 -f Dockerfile .
docker run -p=80:3000 --rm -d cb-ide:v2.0
```

