### Python 3.7 on Centos

**Build image**
```sh
docker build --build-arg CENTOS_VERSION=7.7.1908 --target py37app --tag py37app:latest .
```

**Run image**
```sh
docker run --rm -it py37app:latest
```
