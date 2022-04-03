# Week 11 Homework: Signature Project: Automated Essay Scoring System

# Step 1: Develop and test the solution on a VM

## Create an VM instance on GCP

we will use Ubuntu 20.04 LTS image, make sure there is an external IP, try to use with big size like 50 GB

![Untitled](Week%2011%20Ho%2028cd1/Untitled.png)

## Install common package

- command

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y build-essential python3-pip libxml-parser-perl unzip
sudo apt-get install -y pkg-config libpng-dev libfreetype6-dev freetype2-demos
pip3 install gdown
vi ~/.bashrc
```

- .bashrc content

```
.....
export PATH="$PATH:$HOME/.local/bin"
```

- command

```bash
source ~/.bashrc
```

## Download and unzip the package

```bash
gdown https://drive.google.com/uc\?id\=1RxfZOYyNvzvCf37_vABfJMkohAsEZKtH
unzip rough.zip
wget https://s3.amazonaws.com/models.huggingface.co/bert/bert-large-uncased.tar.gz
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%201.png)

## Install and test ROUGE

```bash
sudo cpan install XML::Parser::PerlSAX
sudo cpan install XML::RegExp
sudo cpan install XML::DOM
cd RELEASE-1.5.5/
./runROUGE-test.pl
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%202.png)

## Install pyrouge

```bash
cd
git clone https://github.com/bheinzerling/pyrouge.git
cd pyrouge
pip3 install -e .
```

# Clone the project and change the hardcode path on [BertParent.py](http://BertParent.py) line 48

```bash
git clone https://github.com/Quan25/flask-summary.git
pwd
vi flask-summary/summarizer/BertParent.py
```

- change content

```bash
# from
self.model = BertModel.from_pretrained('/home/quan/Downloads/bert-large-uncased')
# to
self.model = BertModel.from_pretrained('/home/chen19606/bert-large-uncased.tar.gz')
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%203.png)

## Install flask and related python package

```bash
pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu
pip3 install flask pandas sklearn nltk
pip3 install gensim==3.8.3
pip3 install pytorch-pretrained-bert
pip3 install matplotlib==3.0.1
```

## create punktDownload.py

- command

```bash
vi punktDownload.py
```

- content

```python
import nltk

nltk.download('punkt')
```

- command

```bash
python3 punktDownload.py
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%204.png)

## Run Flask app

```bash
cd flask-summary/
python3 app.py
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%205.png)

## Fork the project and change the hardcode path for the dockerfile

## Install docker on Ubuntu

[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

## Build Docker image with Dockerfile and push to the Dockerhub

- command

```bash
cd
vi Dockerfile
```

- content

```docker
FROM ubuntu:20.04

ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN mkdir -p /home/project/

COPY rough.zip /home/project/
COPY punktDownload.py /home/project/
COPY bert-large-uncased.tar.gz /home/project/

WORKDIR /home/project/
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y build-essential python3-pip libxml-parser-perl unzip git
RUN apt-get install -y pkg-config libpng-dev libfreetype6-dev freetype2-demos

RUN unzip rough.zip
RUN cpan install XML::Parser::PerlSAX
RUN cpan install XML::RegExp
RUN cpan install XML::DOM

WORKDIR /home/project/RELEASE-1.5.5
RUN ./runROUGE-test.pl

WORKDIR /home/project/
RUN git clone https://github.com/bheinzerling/pyrouge.git

WORKDIR /home/project/pyrouge
RUN pip install -e .

WORKDIR /home/project/
RUN git clone https://github.com/tchen0915/flask-summary.git
RUN pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu
RUN pip3 install flask pandas sklearn nltk
RUN pip3 install gensim==3.8.3
RUN pip3 install pytorch-pretrained-bert
RUN pip3 install matplotlib==3.0.1
RUN python3 punktDownload.py

WORKDIR /home/project/flask-summary
EXPOSE 5000
CMD ["python3", "app.py"]
```

- command

```bash
sudo docker build -t tchen0915/aes .
# sudo docker run -p 5000:5000 -t tchen0915/aes
sudo docker login
sudo docker push tchen0915/aes
```

# Step 2: Test your image on the GCP cloudshell

## run the docker image

```bash
docker run -p 5000:5000 -t tchen0915/aes
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%206.png)

## Use web Preview and change the port number to 5000

![Untitled](Week%2011%20Ho%2028cd1/Untitled%207.png)

![Untitled](Week%2011%20Ho%2028cd1/Untitled%208.png)

![Untitled](Week%2011%20Ho%2028cd1/Untitled%209.png)

# Step 3: Deploy this solution on Kubernetes using cloud shell

## Create cluster

```bash
gcloud container clusters create kubia --num-nodes=1 --machine-type=c2-standard-4 --region=us-west1-a
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2010.png)

## Create deployment

- command

```bash
vi aes-deployment.yaml
```

- content

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aes-deployment
spec:
  selector:
    matchLabels:
      app: aes-deployment
  replicas: 1
  template:
    metadata:
      labels:
        app: aes-deployment
    spec:
      containers:
      - name: aes-deployment
        image: tchen0915/aes
        ports:
        - containerPort: 5000
```

- command

```bash
kubectl create -f aes-deployment.yaml
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2011.png)

## Create service

- command

```bash
vi aes-service.yaml
```

- content

```yaml
apiVersion: v1
kind: Service
metadata:
  name: aes-service
spec:
  selector:
    app: aes-deployment
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
```

- command

```bash
kubectl create -f aes-service.yaml
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2012.png)

## Create ingress

- command

```bash
vi aes-ingress.yaml
```

- content

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aes-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - host: aes.wifrost.top
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: aes-service
                port:
                  number: 5000
```

- command

```bash
kubectl create -f aes-ingress.yaml
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2013.png)

## Check status

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2014.png)

## Copy IP to your domain DNS server and create a record

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2015.png)

## Access the domain from the browser

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2016.png)

## Test it with random wiki paragraph (I tried google wiki)

Google LLC is an American multinational technology company that specializes in Internet-related services and products, which include a search engine, online advertising, cloud computing, software, artificial intelligence,[10] quantum computing,[11] and hardware. It has been referred to as the "most powerful company in the world" and one of the world's most valuable brands due to its market dominance, data collection, and technological advantages in the area of artificial intelligence.[12][13][14] It is considered one of the Big Five American information technology companies, alongside Amazon, Apple, Meta, and Microsoft.

Google was founded on September 4, 1998, by Larry Page and Sergey Brin while they were PhD students at Stanford University in California. Together they own about 14% of its publicly listed shares and control 56% of the stockholder voting power through super-voting stock. The company went public via an initial public offering (IPO) in 2004. In 2015, Google was reorganized as a wholly-owned subsidiary of Alphabet Inc.. Google is Alphabet's largest subsidiary and is a holding company for Alphabet's Internet properties and interests. Sundar Pichai was appointed CEO of Google on October 24, 2015, replacing Larry Page, who became the CEO of Alphabet. On December 3, 2019, Pichai also became the CEO of Alphabet.[15]

The company has since rapidly grown to offer a multitude of products and services beyond Google Search, many of which hold dominant market positions. These products address a wide range of use cases including email (Gmail), navigation (Maps), cloud computing (Cloud), web browsing (Chrome), video sharing (YouTube), productivity (Workspace), operating systems (Android), cloud storage (Drive), language translation (Translate), photo storage (Photo), video calling (Meet), smart home (Nest), smartphones (Pixel), wearable technology (Fitbit), gaming (Stadia), music streaming (YouTube Music), video on demand (TV), artificial intelligence (Assistant), machine learning (TensorFlow), AI chips (TPU), and more. The company is also notorious for its vast portfolio of discontinued or replaced products,[16][17] which includes Google+, Reader, Play Music, Nexus, Hangouts, and Inbox by Gmail.

Google is well-known for it highly ambitious technological innovations aimed at solving humanity's biggest problems.[18] Some of these innovations include quantum computing (Sycamore), self-driving cars (Waymo, formerly the Google self-driving project), and smart cities (Sidewalk Labs), transformer models (Google Brain).

Google.com and YouTube.com are the two most visited websites worldwide followed by Facebook and Twitter. Google is also the largest search engine, mapping and navigation service, email provider, office suite, video sharing platform, photo and cloud storage provider, mobile operating system, web browser, ML framework, and AI virtual assistant provider in the world as measured by market share. On the list of most valuable brands, Google is ranked second by Forbes[19] and fourth by Interbrand.[20] It has received significant criticism involving issues such as privacy concerns, tax avoidance, censorship, search neutrality, antitrust and abuse of its monopoly position.

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2017.png)

Click **submit** button

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2018.png)

Click **Grade Students** button

![Untitled](Week%2011%20Ho%2028cd1/Untitled%2019.png)

# Reference

- Github: [https://github.com/Quan25/flask-summary](https://github.com/Quan25/flask-summary)