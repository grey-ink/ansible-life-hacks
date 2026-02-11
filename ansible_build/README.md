build docker image for nsible
=============================================
Чтобы собрать образ с нужными компонентами, нужно добавить или раскомментировать их в следующих файлах:
- OS packages: apk.txt
- python dependencies: requirements.txt
- ansible-galaxy collections: requirements.yml

ansible-galaxy usage
--------------------------------------------
Нужно заменить в Dockerfile
```commandline
  && rm -rf /root/.cargo
```
на
```commandline
    && rm -rf /root/.cargo \
    && ansible-galaxy collection install -U -r ./requirements.yml -v
```
