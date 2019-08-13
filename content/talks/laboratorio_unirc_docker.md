---
title: 'Laboratorio pratico di Docker 🇮🇹'
date: "2019-05-20T14:30:00"
categories: ["workshop"]
tags: ["docker", "university", "java"]
---
class: center, middle
name: start

# Laboratorio pratico di Docker ![:i](fab fa-docker)

---
class: center, middle
## Sono

# Danilo Spinella

---
class: left, top

### Obbiettivi

## Concetti sui Containers

## Setup ambiente Docker

## Avvio di containers

## Applicazione Java su un container

---
class: center, middle

### Parte 1:

## Concetti sui Containers

---

class: center, middle
### Macchina virtuale

## Emula il comportamento di una macchina fisica</br>attraverso l'assegnamento di risorse hardware
---
class: center, middle
![](/img/laboratorio_unirc_docker/MacchineVirtuali.png)
---
class: left, top
### Vantaggi

--
## Isolamento dall'Host

--
## Guest OS indipendente

--
## Portabilità del sistema

---
class: left, top
### Svantaggi

--
## Inefficenza

--
## Risorse non condivise

--
## Avvio lento

---
class: center, middle

### Containers

## Un'unità standardizzata di software

---
class: center, middle

![:resize 1000](/img/laboratorio_unirc_docker/VirtualMachinesVSContainers.png)
---
class: left, top

### Vantaggi

--

## Basso overhead
--

## Gestione risorse

--

## Scalabilità

---
class: center, middle

### Parte 2:

## Setup ambiente Docker

---
class: center, middle

### Per Windows

## Avviare una macchina virtuale Linux

---

### Per Ubuntu 18.04

```bash
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

---
class: center, middle

## Proviamo docker:

```bash
sudo docker run hello-world
```

---
class: center, middle

# Parte 3:

## Avvio containers

---

## Controlliamo le immagini scaricate:

```bash
sudo docker image ls
```

## Controlliamo i container avviati:

```bash
sudo docker container ls
```

---

## Scarichiamo l'immagine di Ubuntu:

```bash
sudo docker pull ubuntu:18.04
```

## Avviamo il container usando l'immagine appena scaricata e avviamo una shell:

```bash
sudo docker run -i -t ubuntu:18.04 /bin/bash
```

## Controlliamo di trovarci nel container

```bash
ls
```

---

## Controlliamo la versione del kernel usata dal container:

```bash
uname -a
```

## Torniamo all'host:

```bash
exit
```

## Controlliamo la versione del kernel dell'host:

```bash
uname -a
```

---
class: center, middle

## Dimensione Immagine Ubuntu: ~28MB

## Dimensione Iso Ubuntu: ~1.9GB

---
class: center, middle

### Parte 4:

## Applicazione Java

---

### Scriviamo una semplice applicazione Java

```
DockerExample.java
```

```java
public class DockerExample {
  public static void main(String[] args) {
    System.out.println("Il programma è stato avviato!");
  }
}
```

## e creiamo un Dockerfile:

```
Dockerfile
```

```Dockerfile
FROM java:8  
COPY . /work
WORKDIR /work
RUN javac DockerExample.java
CMD [ "java", "DockerExample" ]
```

---

## Creiamo un'immagine:

```bash
sudo docker build -t example .
```

## Avviamo il container

```bash
sudo docker run example
```

---
class: center, middle

## Grazie per l'attenzione

# Ci sono domande?

---
template: start

