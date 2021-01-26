# Ein Intro zu JupyterHub extensions

## Einführung



<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/1200px-Jupyter_logo.svg.png" alt="Project Jupyter – Wikipedia" style="zoom:10%;" />					 

Um Jupyter auf einem Multi-User-Server zu benutzen, verwendet man *JupyterHub*. Hier können Nutzer verwaltet und Umgebungen konfiguriert werden. Es gibt zwei mögliche Interfaces,  welche auf JupyterHub aufgesetzt werden können: *JupyterLab* und *The Jupyter notebook*.  

Wenn man *JupyterLab* auf einem System installiert, welches *JupyterHub* betreibt, so ist *JupyterLab* direkt unter der `/lab`  URL erreichbar. *The Jupyter notebook* ist weiterhin unter `/tree`  verfügbar. Der default kann ganz einfach im *JupyterHub* config file (`jupyterhub_config.py`) geändert werden.

In `jupyterhub_config.py`  kann man Dinge wie Datenbank Konfigurationen oder User Management beeinflussen. Will man aber Details wie Benutzeroberflächen, git Integration oder andere Kleinigkeiten verändern, gibt es eine Vielzahl an Extensions. 

![Image for post](https://miro.medium.com/max/2199/1*P0B1z-38LkFEXZuA5RER8A.png)



## Beispiele

### Version Control via Git

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Git-logo.svg/1200px-Git-logo.svg.png" alt="Git – Wikipedia" style="zoom:15%;" />	



Es ist sehr schwierig diffs in jupyter notebooks zu erkennen, wenn man git mit dem klassischen Terminal bedient, da die jupyter notebooks (.ipynb) als JSON file abgespeichert werden. Um diesem Problem entgegenzuwirken, wurden einige open source Lösungen entwickelt. Ich stelle hier eine Lösung für JupyterLab mit dem Namen [jupyterlab-git](https://github.com/jupyterlab/jupyterlab-git) vor.  Für The Jupyter notebook habe ich noch keine elegante Lösung gefunden. Für mmN unschönere Lösungen könnte man [diesen Blog](https://towardsdatascience.com/version-control-with-jupyter-notebooks-f096f4d7035a) in Betracht ziehen. 

##### Installation

Die Installation erfolgt ganz einfach über pip: 

```bash
pip install --upgrade jupyterlab jupyterlab-git
jupyter lab build
```

##### Settings

Konfiguriert kann die git Nutzung unter JupyterLab's advanced settings Editor und in JupyterHubs config file (`jupyterhub_config.py`)  

##### Demo

![preview.gif](https://github.com/jupyterlab/jupyterlab-git/blob/master/docs/figs/preview.gif?raw=true)



### Themes



## Quellen



https://towardsdatascience.com/version-control-with-jupyter-notebooks-f096f4d7035a (git)

https://github.com/jupyterlab/jupyterlab-git (git)

https://jupyterlab.readthedocs.io/en/stable/user/jupyterhub.html (jupyterLab Integration)

https://jupyterlab.readthedocs.io/en/stable/user/extensions.html (notebook extensions)