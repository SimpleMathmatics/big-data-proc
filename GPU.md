# GPU-Sharing Über Mehrere Container 



## Problemstellung

Bisweilen gibt es keinen offiziellen Weg, um Containerübergreifend (und Podübergreifend) auf eine einzige GPU zuzugreifen und sich diese zu teilen. Normalerweise wird die GPU geblockt, sobald sie von einem Pod in Benutzung ist, und erst wieder freigegeben, wenn dieser verschwindet. Ich stelle hier eine Methode vor, welche dieses Problem umgeht. 



## Ausgangssituation

Es sollen mehrere Nvidia Tesla T4 Grafikkarten über Kubernetes geteilt werden (etwa um von einem JupyterLab darauf zugreifen zu können). Diese werden mit dem aktuellen Treiber und Cuda 11 betrieben. 



## Lösung

Nach einiger Recherche bin ich aber auf [dieses Repo](https://github.com/AliyunContainerService/gpushare-scheduler-extender/) gestoßen, welche es über Umwege möglich macht dies zu erreichen. Für mich schaut das vielversprechen aus. Im Grunde werden hier zwei wichtige Komponenten eingeführt, ohne in die Kubernetesstruktur einzugreifen: 

![img](https://github.com/AliyunContainerService/gpushare-scheduler-extender/raw/master/docs/designs/arch.jpg)



- Plugin, um alle GPU Resourcen zu binden

- Sceduler Extender, welcher die Resourcen zuteilt.  Dieser erweitert den Kubernetes Sceduler (Siehe folgende Grafik)

  ![filter.jpg](https://github.com/AliyunContainerService/gpushare-scheduler-extender/blob/master/docs/designs/filter.jpg?raw=true)

Nach erfolgreicher Installation kann man dann für jede Node entscheiden, ob sie auf die Resourcen zugreifen kann.

#### Voraussetzungen

Folgende Vorraussetzungen sollten erfüllt sein: 

- Kubernetes 1.11+ (1.17.5 im Einsatz)
- golang 1.10+ (fehlt)
- NVIDIA drivers ~= 361.93 (Aktuellesten Treiber mit Cuda 11)
- Nvidia-docker version > 2.0 (ist installiert)
- Docker configured with Nvidia as the default runtime (ist so konfiguriert)

Im Vorhandenen Setup fehlt also nur eine golang Installation. Golang kann wie folgt installiert werden:

```bash
# download latest version in archinve file (64-bit)
$ wget -c https://golang.org/dl/go1.15.2.linux-amd64.tar.gz   [64-bit]

#  check the integrity of the tarball
$ shasum -a 256 go1.7.3.linux-amd64.tar.gz

# extract the tar archive files into /usr/local
$ sudo tar -C /usr/local -xvzf go1.15.2.linux-amd64.tar.gz

# set up your Go workspace by creating a directory ~/go_projects
$ mkdir -p ~/go_projects/{bin,src,pkg}
$ cd ~/go_projects

# add /usr/local/go/bin to the PATH
$ export  PATH=$PATH:/usr/local/go/bin

# set the values of GOPATH and GOBIN
$ export GOPATH="$HOME/go_projects"
$ export GOBIN="$GOPATH/bin"
```

 

#### Installation

Wir folgenden [diesen 5 Schritten](https://github.com/AliyunContainerService/gpushare-scheduler-extender/blob/master/docs/install.md) um das GPU-sharing einzurichten:

1. **Deploy GPU share scheduler extender**

```bash
$ cd /etc/kubernetes/

$ curl -O https://raw.githubusercontent.com/AliyunContainerService/gpushare-scheduler-extender/master/config/scheduler-policy-config.json

$ cd /tmp/

$ curl -O https://raw.githubusercontent.com/AliyunContainerService/gpushare-scheduler-extender/master/config/gpushare-schd-extender.yaml

$ kubectl create -f gpushare-schd-extender.yaml
```



2. **Modifiziere Sceduler config**

   unter command füge folgendes hinzu:

   ```
   - --policy-config-file=/etc/kubernetes/scheduler-policy-config.json
   ```

   Füge volumes hinzu:

   ```yaml
   - mountPath: /etc/kubernetes/scheduler-policy-config.json
     name: scheduler-policy-config
     readOnly: true
     
   - hostPath:
         path: /etc/kubernetes/scheduler-policy-config.json
         type: FileOrCreate
     name: scheduler-policy-config
   ```

   

3. **Deploy device plugin**

   ```bash
   $ wget https://raw.githubusercontent.com/AliyunContainerService/gpushare-device-plugin/master/device-plugin-rbac.yaml
   
   $ kubectl create -f device-plugin-rbac.yaml
   
   $ wget https://raw.githubusercontent.com/AliyunContainerService/gpushare-device-plugin/master/device-plugin-ds.yaml
   
   $ kubectl create -f device-plugin-ds.yaml
   ```

   

4.  **Gebe jeder nötigen Node die Möglichkeit auf die GPU-Ressourcen  zuzugreifen**

   ```bash
   $ kubectl label node <target_node> gpushare=true
   ```

   

5. **Installiere cubectl extension**

   ```bash
   $ cd /usr/bin/
   
   $ wget https://github.com/AliyunContainerService/gpushare-device-plugin/releases/download/v0.3.0/kubectl-inspect-gpushare
   
   $ chmod u+x /usr/bin/kubectl-inspect-gpushare
   ```

   

## Quellen

https://github.com/AliyunContainerService/gpushare-scheduler-extender/

https://code.tubitv.com/jupyterhub-on-kubernetes-da8940488529

https://github.com/AliyunContainerService/gpushare-scheduler-extender/blob/master/docs/install.md

https://www.scaleway.com/en/docs/setup-configure-jupyter-notebook-gpu-instance/

https://blog.paperspace.com/jupyter-notebook-with-a-gpu-the-easy-way/

https://medium.com/rapids-ai/setting-up-gpu-data-science-environments-for-hackathons-cdb52e7781a5

https://www.tecmint.com/install-go-in-linux/

