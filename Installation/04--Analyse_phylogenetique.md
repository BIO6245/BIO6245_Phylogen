# Installation de programmes d'analyse phylogénétique

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

