# Zadania lab 04

## 1. Endpoint `GET /health`

```console
$ curl http://localhost:5000/health
{"status":"ok"}
```

```console
$ docker logs backend
Wczytano 89 hashy z /app/data/leaked-sha1.txt
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000
Press CTRL+C to quit
172.17.0.1 - - [12/Nov/2026 14:55:37] "GET /health HTTP/1.1" 200 -
```

## 2. Kontener diagnostyczny

```
docker run --rm --network breach-net probe
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0{"status":"ok"}
100     16 100     16   0      0  11396      0                              100     16 100     16   0      0  11157      0                              100     16 100     16   0      0  10943      0                              0
backend: OK

docker run --rm probe
curl: (6) Could not resolve host: backend (Domain name not found)
backend: UNREACHABLE
```

Nazwa backend nie działa bez `--network`, ponieważ tylko w danej sieci Docker daje serwer DNS, który przekierowuje nazwy kontenerów na ich wewnętrzne adresy IP. Poza tą siecią taka nazwa hosta nie istnieje.

## 3. Trwałość danych

Warstwa zapisywalna:

```
[main] breach-checker >> docker start backend
backend
[main] breach-checker >> docker exec backend sh -c "echo notatka > /tmp/notatka.txt"
[main] breach-checker >> docker restart backend
backend
[main] breach-checker >> docker exec backend cat /tmp/notatka.txt
notatka
[main] breach-checker >> docker rm -f backend
backend

[main] breach-checker >> docker run -d --name backend -v ./data:/app/data $b:v2
ebe1aa17d10a0898fade401e4017a2c2b25278bed6fe5a8b2941d7f8ca561446
[main] breach-checker >> docker exec backend cat /tmp/notatka.txt
cat: /tmp/notatka.txt: No such file or directory
```

Wolumen:

```
[main] breach-checker >> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt"
[main] breach-checker >> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt; cat /dane/historia.txt"
Thu Nov 12 15:08:33 UTC 2026
Thu Nov 12 15:08:45 UTC 2026
[main] breach-checker >> docker volume ls
DRIVER    VOLUME NAME
local     breach-dane
[main] breach-checker >> docker volume rm breach-dane
breach-dane
```

Plik w /tmp przetrwał restart ale nie rm, ponieważ zestopowanie/zrestartowanie kontenera nie usuwa jego warstwy tymczasowej, jednak rm - już tak. historia.txt przetrwała, ponieważ była zapisana na wolumenie.

Lista wycieków z wcześniejszych zadań przetrwała, bo znajdowała się na wolumenie.
