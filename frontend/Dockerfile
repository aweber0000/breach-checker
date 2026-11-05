FROM nginx:alpine
LABEL autor="Artur Weber" data="2026-11-05" maintainer="imie.nazwisko@example.com"
RUN apk add --no-cache nodejs npm && npm install -g htmlhint
COPY . /usr/share/nginx/
RUN htmlhint /usr/share/nginx/html/index.html
#RUN date > /usr/share/nginx/html/build.txt
EXPOSE 80
