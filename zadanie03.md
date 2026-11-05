## 1. Etykiety

```console
$ docker inspect --format '{{json .Config.Labels}}' artur-weber-frontend:latest
{"autor":"Artur Weber","data":"2026-11-05","maintainer":"imie.nazwisko@example.com"}
```

```console
$ docker history artur-weber-frontend:latest
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
aae623a1c359   About a minute ago   EXPOSE [80/tcp]                                 0B        buildkit.dockerfile.v0
<missing>      About a minute ago   RUN /bin/sh -c htmlhint /usr/share/nginx/htm…   0B        buildkit.dockerfile.v0
<missing>      About a minute ago   COPY . /usr/share/nginx/html # buildkit         1.54kB    buildkit.dockerfile.v0
<missing>      About a minute ago   RUN /bin/sh -c apk add --no-cache nodejs npm…   89.2MB    buildkit.dockerfile.v0
<missing>      About a minute ago   LABEL autor=Artur Weber data=2026-11-05 main…   0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          RUN /bin/sh -c set -x     && apkArch="$(cat …   50.1MB    buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV ACME_VERSION=0.4.1                          0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV NJS_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV NJS_VERSION=1.0.2                           0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          COPY 30-tune-worker-processes.sh /docker-ent…   4.62kB    buildkit.dockerfile.v0
<missing>      2 weeks ago          COPY 20-envsubst-on-templates.sh /docker-ent…   3.03kB    buildkit.dockerfile.v0
<missing>      2 weeks ago          COPY 15-local-resolvers.envsh /docker-entryp…   389B      buildkit.dockerfile.v0
<missing>      2 weeks ago          COPY 10-listen-on-ipv6-by-default.sh /docker…   2.14kB    buildkit.dockerfile.v0
<missing>      2 weeks ago          COPY docker-entrypoint.sh / # buildkit          1.62kB    buildkit.dockerfile.v0
<missing>      2 weeks ago          RUN /bin/sh -c set -x     && addgroup -g 101…   4.42MB    buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV DYNPKG_RELEASE=1                            0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV PKG_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ENV NGINX_VERSION=1.31.8                        0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      2 weeks ago          ADD alpine-minirootfs-3.24.3-x86_64.tar.gz /…   8.43MB    buildkit.dockerfile.v0
```

Warstwy od `LABEL autor=...` wzwyż są z naszego obrazu; wszystko poniżej pochodzi z obrazu bazowego.

Niektóre warstwy mają rozmiar 0B, ponieważ nie dodają plików do finalnego obrazu (np. ENV ustawiające
zmienne środowiskowe lub LABEL ustawiające etykiety).

Widać tylko moją wartość `maintainer`, ponieważ oryginalna została nadpisana.

## 2. Komenda RUN

```console
$ docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED         STATUS                     PORTS     NAMES
abe0c76d2f35   artur-weber-frontend:v2   "/docker-entrypoint.…"   2 minutes ago   Exited (0) 3 seconds ago             objective_wu
4870d6806a87   artur-weber-frontend:v2   "/docker-entrypoint.…"   2 minutes ago   Exited (0) 2 minutes ago             youthful_brahmagupta
```

build.txt w pierwsym kontenerze:

```
Thu Nov  5 15:05:13 UTC 2026
```

build.txt w drugim kontenerze:

```
Thu Nov  5 15:05:13 UTC 2026
```

Są one takie same, ponieważ plik build.txt jest generowany podczas budowania obrazu, nie podczas jego uruchamiania; komenda RUN wykonuje się w trakcie budowania, więc data w pliku pokazuje czas builda.

## Nadpisanie CMD

```nginx
$ docker run --rm artur-weber-frontend:v2 cat /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # ...
}
```

Wrzucamy pliki do /usr/share/nginx/html, ponieważ jest to domyślna ścieżka, w której domyślna konfiguracja nginxa szuka plików do zaserwowania.

## 4. Błędne COPY

```console
$ docker run -d -p 8080:80 artur-weber-frontend:v3
8c4a014cabe823b46e7af1cbf8c31064a6b02a7222e08d5ad92397c3e3be0dc2
$ docker exec 8c4a014cabe823b46e7af1cbf8c31064a6b02a7222e08d5ad92397c3e3be0dc2 ls /usr/share/nginx
html
index.html
script.js
style.css
$ docker exec 8c4a014cabe823b46e7af1cbf8c31064a6b02a7222e08d5ad92397c3e3be0dc2 ls /usr/share/nginx/html
50x.html
index.html
```

Domyślnie w folderze /usr/share/nginx/html znajduje się domyślny index.html nginxa; htmlhint sprawdza jego zamiast naszego pliku.

## 5. Usunięcie tagu a usunięcie obrazu

```console
$ docker images
                                                                        i Info →   U  In Use
IMAGE                         ID             DISK USAGE   CONTENT SIZE   EXTRA
artur-weber-frontend:latest   893cffa2065a        152MB             0B        
artur-weber-frontend:v1       aae623a1c359        152MB             0B        
artur-weber-frontend:v2       4b1e1ee3e30d        152MB             0B        
artur-weber-frontend:v3       893cffa2065a        152MB             0B        

$ docker rmi artur-weber-frontend:latest
Untagged: artur-weber-frontend:latest

$ docker images
                                                                        i Info →   U  In Use
IMAGE                     ID             DISK USAGE   CONTENT SIZE   EXTRA
artur-weber-frontend:v1   aae623a1c359        152MB             0B        
artur-weber-frontend:v2   4b1e1ee3e30d        152MB             0B        
artur-weber-frontend:v3   893cffa2065a        152MB             0B        

$ docker rmi artur-weber-frontend:v3
Untagged: artur-weber-frontend:v3
Deleted: sha256:893cffa2065a49fb0cd3537b5757801937d8e5340d7630a226ab37e2c0017415

$ docker images
                                                                        i Info →   U  In Use
IMAGE                     ID             DISK USAGE   CONTENT SIZE   EXTRA
artur-weber-frontend:v1   aae623a1c359        152MB             0B        
artur-weber-frontend:v2   4b1e1ee3e30d        152MB             0B        
```

`docker rmi` usuwa obraz z dysku, gdy nie jest on dostępny pod żadnym tagiem.
