### Dockerfile
```dockerfile
FROM python:3.12-alpine

LABEL org.opencontainers.image.title="ansible for infrastructure as code"

WORKDIR /tmp

COPY ./*.txt .

RUN apk add --no-cache $(cat ./apks.txt) \
    && apk --no-cache add --virtual \
        build-dependencies \
        build-base \
        libffi-dev \
        musl-dev \
        cargo \
        gcc \
    && update-ca-certificates \
    && pip3 install --upgrade pip cffi cryptography wheel \
    && pip3 install -r ./requirements.txt \
    && apk del build-dependencies \
    && rm -rf /var/cache/apk/* \
    && rm -rf /root/.cache/pip \
    && rm -rf /root/.cargo
RUN mkdir /ansible \
    && mkdir -p /etc/ansible \
    && echo 'localhost' > /etc/ansible/hosts \
    && ln -s /usr/local/lib/python3.12 /usr/local/lib/python3.9 \
    && ln -s /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.9/dist-packages

WORKDIR /ansible

CMD ["ansible-playbook" "--version"]
```

### Списки зависимостей
Нужно положить в директорию рядом с Dockerfile или поменять в Dockerfile пути к ним
#### requirements.txt
```text
ansible-lint<25
ansible>=9,<10
python-gitlab
cryptography
MarkupSafe
kubernetes
jmespath
netaddr
mitogen<=0.3.24
jinja2
hvac
pbr
ruamel.yaml.clib
ruamel.yaml
yamllint
```
#### apks.txt
```text
ca-certificates
sshpass
openssh
openssl
unzip
rsync
sudo
curl
wget
tar
git
```