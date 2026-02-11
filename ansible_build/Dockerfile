FROM python:3.12-alpine
# Если нужно брать образ из внутреннего репозитория
#FROM internal_registry_hostname/image_repo_name/python:3.12-alpine

LABEL org.opencontainers.image.title="ansible for infrastructure as code"

# Если нужно брать пакеты python из внутреннего репозитория
#ENV PIP_INDEX_URL=https://internal_repo_hostname/repository/python_repo_name/simple/

WORKDIR /tmp

COPY ./*.txt .

# Если нужно использовать внутренний репозиторий пакетов alpine
#RUN echo "#/media/cdrom/apks" > /etc/apk/repositories \
#    && echo "https://internal_repo_hostname/repository/alpine_package_repo/alpine/latest-stable/main" >> /etc/apk/repositories \
#    && echo "https://internal_repo_hostname/repository/alpine_package_repo/latest-stable/community" >> /etc/apk/repositories \
#    && apk update \
#    && apk add --no-cache $(cat ./apk.txt) \
RUN apk add --no-cache $(cat ./apk.txt) \
    && apk --no-cache add --virtual \
        build-dependencies \
        build-base \
        libffi-dev \
        musl-dev \
        cargo \
        gcc \
# Если нужно добавить внутренний root CA
#    && curl root_ca_file_url/root_ca_file_name -o /usr/local/share/ca-certificates/root_ca_file_name \
#    && update-ca-certificates \
    && pip3 install --upgrade pip cffi cryptography wheel \
    && pip3 install -r ./requirements.txt \
    && apk del build-dependencies \
    && rm -rf /var/cache/apk/* \
    && rm -rf /root/.cache/pip \
    && rm -rf /root/.cargo
RUN mkdir /ansible \
    && mkdir -p /etc/ansible \
    && echo 'localhost' > /etc/ansible/hosts \
# Если где-то в ansible захардкожен старый путь к плагинам
#    && ln -s /usr/local/lib/python3.12 /usr/local/lib/python3.9 \
#    && ln -s /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.9/dist-packages

WORKDIR /ansible

CMD ["ansible-playbook" "--version"]
