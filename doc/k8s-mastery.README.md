
read ~/microservice/k8s-mastery/doc/Kubernetes-in-3Hours.pdf

## build and deploy k8s-mastery microservices (w/o docker)

** component = sa-frontend (VIEW)
    port = 80

$ cd ~/microservices/k8s-mastery/sa-frontend
$ npm install

$ npm start  # react runs on localhost:3000

$ npm run build

*** deploy to nginx

$ cd ~/microservices/k8s-mastery/sa-frontend/build
$ sudo cp -r * /var/www/html

$ sudo systemctl reload nginx
$ sudo systemctl status nginx

check url=http://localhost:8001/

** component = sa-webapp (CONTROLLER)
    port = 8002

$ cd ~/microservices/k8s-mastery/sa-webapp/
$ mvn install

$ cd ~/microservices/k8s-mastery/sa-webapp/target
$ java -jar sentiment-analysis-web-0.0.1-SNAPSHOT.jar --server.port=8002 --sa.logic.api.url=http://localhost:5000

or define sa.logic.api.url=http://localhost:5000
server.port=8002
 in sa-webapp/src/main/resources/application.properties

open url=http://localhost:8002/
see Whitelabel Error Page

** component = sa-logic (MODEL/SERVICE)
    port = 5000

$ cd ~/microservices/k8s-mastery/sa-logic/sa
$ python -m pip install -r requirements.txt 
$ python -m textblob.download_corpora

$ python sentiment_analysis.py

open ulr=http://localhost:5000/
see "Welcome to Flask" msg defined for route "/"

$ curl http://localhost:8001
curl: (7) Failed to connect to localhost port 8001: Connection refused

$ sudo ufw allow 8001
$ sudo ufw allow 8002
$ sudo ufw status

* need to restart ufw after adding rules
$ sudo systemctl stop ufw
$ sudo systemctl start ufw
$ sudo systemctl status ufw

## build and deploy k8s-mastery microservices (w/ docker)

$ cd ~/microservices/k8s-mastery/sa-frontend
** login to dockerhub
$ docker login -u="w3gong" -p="web_boy27514"

** vi .dockerignore
add 
    node_modules
    src
    public

** build
$ docker build -f Dockerfile -t w3gong/sentiment-analysis-frontend .

** push to dockerhub
$ docker push w3gong/sentiment-analysis-frontend

** login to https://hub.docker.com/ and verified that above image exists

** pull from docker hub and run it locally
$ docker pull w3gong/sentiment-analysis-frontend
$ docker run -d -p 80:80 w3gong/sentiment-analysis-frontend

Error starting userland proxy: listen tcp 0.0.0.0:80: bind: address already in use.

$ docker run -d -p 8001:80 w3gong/sentiment-analysis-frontend

$ docker ps
e3ba39719379

** stop docker
$ docker stop e3ba39719379

** build webapp container
$ cd ../sa-webapp

$ docker build -t w3gong/sentiment-analysis-web-app .

$ docker push w3gong/sentiment-analysis-web-app

** build sa-logic container

$ cd ../sa-logic
$ docker build -t w3gong/sentiment-analysis-logic .
$ docker push w3gong/sentiment-analysis-logic

$ docker run -d -p 8003:5000 w3gong/sentiment-analysis-logic

$ docker inspect daad50d5a292
"IPAddress": "172.17.0.3"


$ docker run -d -p 8002:8080 -e SA_LOGIC_API_URL='http://172.17.0.3:5000' w3gong/sentiment-analysis-web-app  


## build and deploy k8s-mastery microservices (MicroK8s)