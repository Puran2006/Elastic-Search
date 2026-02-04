# Elastic Search Installation

## Installing Windows Linux
I am installing it normally using docker in the wsl linux platform I installed in my windows

```powershell
wsl –install
```
Run the above command in windows powershell and set your username and password

## Install Elastic Search via Docker
Opne the Ubuntu you just installed and update the apt-packages
```bash
 sudo apt update
 sudo apt upgrade -y
```
Check whether docker is working or not
```bash
docker --version
```
If docker is not installed go and install docker, its so easy you would find it anywhere on the internet....

Run the following command to get the elastic search using docker:
``` bash
docker run -p 127.0.0.1:9200:9200 -d --name elasticsearch   -e "discovery.type=single-node"   -e "xpack.security.enabled=false"   -e "xpack.license.self_generated.type=trial"   -v "elasticsearch-data:/usr/share/elasticsearch/data"   docker.elastic.co/elasticsearch/elasticsearch:9.2.3
```

Now run the following commands to create a enviroment using python : 
```bash
 mkdir elastic_search
 cd elastic_search/

 sudo apt install python3.12-venv

 python3 venv elastic
 source elastic/bin/activate
```
Now the enviroment should have been activated 

Lets install the elastic search using pip
```bash
pip install elasticsearch
```
Later while nodes with kibana you need to a docker compose method
```bash
mkdir ~/elastic-search/docker
cd ~/elastic-search/docker
vi docker-compose.yml
```
Copy paste this text 
```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es01
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kb01
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  es_data:

```
```bash
docker compose up -d
# to verify the cotainers
docker ps
```
Now you can access the kibana at the localhost:5610 

Now in the tutorial, the code is written in ipynb files, but I am working in linux, so Install the jupyterlab in terminal

```bash
pip install notebook jupyterlab
```
```bash
jupyter lab
```
got to the **localhost:8880**




