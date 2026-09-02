# Installation de programmes d'analyse de génétique des populations

PAML v1.3.1:  
```bash
cd /opt
sudo git clone https://github.com/millanek/fineRADstructure.git
cd fineRADstructure
sudo ./configure
sudo autoreconf -fi # nécessaire car le makefile de base contenait une erreur
sudo make


```

Dsuite:  
```bash
cd /opt
sudo git clone https://github.com/millanek/Dsuite.git
cd Dsuite
sudo make
cd utils
sudo python3 setup.py install --user --prefix=

## test installation
/opt/Dsuite/Build/Dsuite --help

```