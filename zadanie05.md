# Laboratorium 05

## 1. Healthcheck backendu

```
$ docker compose up -d
[+] up 4/4
 ✔ Network breach-checker_default      Created                                  0.0s
 ✔ Container breach-checker-db-1       Healthy                                  5.7s
 ✔ Container breach-checker-backend-1  Healthy                                 11.3s
 ✔ Container breach-checker-frontend-1 Started                                 11.4s
```

```
$ docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED              STATUS                        PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    About a minute ago   Up About a minute (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         About a minute ago   Up About a minute (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   About a minute ago   Up About a minute             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

## 2. Trwałość bazy

```
[main] breach-checker >> docker compose exec db psql -U breach -d breachdb -c "INSERT INTO leaked_hashes VALUES ('bc45408273d7e1914b741c3407b0b559764f7876');"
INSERT 0 1
[main] breach-checker >> ./check 
{"found":true}
[main] breach-checker >> docker compose down
[+] down 4/4
 ✔ Container breach-checker-frontend-1 Removed                                  0.2s
 ✔ Container breach-checker-backend-1  Removed                                 10.1s
 ✔ Container breach-checker-db-1       Removed                                  0.1s
 ✔ Network breach-checker_default      Removed                                  0.1s
[main] breach-checker >> docker compose up -d
[+] up 4/4
 ✔ Network breach-checker_default      Created                                  0.0s
 ✔ Container breach-checker-db-1       Healthy                                  5.7s
 ✔ Container breach-checker-backend-1  Healthy                                 11.3s
 ✔ Container breach-checker-frontend-1 Started                                 11.5s
[main] breach-checker >> ./check 
{"found":true}
[main] breach-checker >> docker compose down -v
[+] down 5/5
 ✔ Container breach-checker-frontend-1 Removed                                  0.2s
 ✔ Container breach-checker-backend-1  Removed                                 10.1s
 ✔ Container breach-checker-db-1       Removed                                  0.1s
 ✔ Volume breach-checker_pgdata        Removed                                  0.0s
 ✔ Network breach-checker_default      Removed                                  0.1s
[main] breach-checker >> docker compose up -d
[+] up 5/5
 ✔ Network breach-checker_default      Created                                  0.0s
 ✔ Volume breach-checker_pgdata        Created                                  0.0s
 ✔ Container breach-checker-db-1       Healthy                                  5.7s
 ✔ Container breach-checker-backend-1  Healthy                                 11.3s
 ✔ Container breach-checker-frontend-1 Started                                 11.5s
[main] breach-checker >> ./check 
{"found":false}
```

Skrót przetrwał `down`, ponieważ `down` jedynie usuwa kontenery, a nie wolumeny. Flaga `-v` usuwa również wolumeny - w tym wolumen z danymi bazy danych używanej przez backend do szukania hashy. W bazie znów jest 89 skrótów, bo odtworzył ją z domyślnego init.sql + pliku z hashami.

## 3. Skalowanie

Przed restartem:

```
backend-1  | 127.0.0.1 - - [19/Nov/2026 15:02:46] "GET /health HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:02:51] "POST /check HTTP/1.1" 200 -
```

Po restarcie:

```
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-2  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-3  | 127.0.0.1 - - [19/Nov/2026 15:04:17] "GET /health HTTP/1.1" 200 -
backend-2  | check: hash=bc454...
backend-3  | 127.0.0.1 - - [19/Nov/2026 15:04:22] "GET /health HTTP/1.1" 200 -
backend-3  | check: hash=bc454...
backend-3  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-3  | check: hash=bc454...
backend-3  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
backend-3  | check: hash=bc454...
backend-3  | 172.18.0.6 - - [19/Nov/2026 15:04:25] "POST /check HTTP/1.1" 200 -
```

```
docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                    PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    13 seconds ago   Up 7 seconds (healthy)    5000/tcp
breach-checker-backend-2    breach-checker-backend    "python app.py"          backend    13 seconds ago   Up 7 seconds (healthy)    5000/tcp
breach-checker-backend-3    breach-checker-backend    "python app.py"          backend    13 seconds ago   Up 7 seconds (healthy)    5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         13 seconds ago   Up 12 seconds (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   13 seconds ago   Up 1 second               0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

```
docker compose exec frontend nslookup backend
Server:		127.0.0.11
Address:	127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:	backend
Address: 172.18.0.5
Name:	backend
Address: 172.18.0.4
Name:	backend
Address: 172.18.0.3
```

Przed restartem frontendu ruch trafiał do jednej repliki, ponieważ pamiętała ona tylko adres pierwszego backendu pod nazwą hosta "backend"; restart zaktualizował te dane.
Skalowanie byłoby niemożliwe, gdyby backend nadal miał otwarte porty na hoście, bo doszłoby do kolizji - killka kontenerów nie może współdzielić tego samego portu na hoście.

