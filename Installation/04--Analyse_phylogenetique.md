# Installation de programmes d'analyse phylogénétique

## Parcimonie

PAUP\* v.169 (ne fonctionne pas car demande libgfortran.so.4 qui n'est plus disponible sous Ubuntu 
24.04):  
```bash
cd /opt
sudo wget http://phylosolutions.com/paup-test/paup4a169_ubuntu64.gz
sudo gunzip paup4a169_ubuntu64.gz
sudo chmod +x paup4a169_ubuntu64

## tester paup
/opt/paup4a169_ubuntu64 -h

```

PAUP\* v.168 (semble fonctionner sous Ubuntu 24.04):  ```bash
cd /opt
sudo wget http://phylosolutions.com/paup-test/paup4a168_centos64.gz
sudo gunzip paup4a168_centos64.gz
sudo chmod +x paup4a168_centos64

## tester paup
/opt/paup4a168_centos64 -h

```


## Maximum de vraisemblance

FastTree:
```bash
sudo apt-get update
sudo apt-get install fasttree

```

RAxML:
```bash
cd /opt
sudo git clone https://github.com/stamatak/standard-RAxML.git
sudo mv standard-RAxML RAxML-8.2.12
cd RAxML-8.2.12
sudo make -f Makefile.SSE3.PTHREADS.gcc

```

