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

# Elastic Search Concepts

## index

Think of an index like a database table in MySQL, but optimized for search.

- **In MySQL** : you have a database → table → rows → columns

- **In Elasticsearch** : you have a cluster → index → documents → fields

Data inside an index are documents (JSON objects). Example document in parts index:
```json
{
  "part_name": "Brake Pad",
  "brand": "Bosch",
  "price": 1200,
  "in_stock": true
}
```
Now in the tutorial, the code is written in ipynb files, but I am working in linux, so Install the jupyterlab in terminal

```bash
pip install notebook jupyterlab

# optional
pip install numpy pandas
```
Start the elastic search docker : 
s
```bash
docker start elasticsearch
```

Run Jupyter inside this environment:

```bash
jupyter lab
```
got to the **localhost:8880**



