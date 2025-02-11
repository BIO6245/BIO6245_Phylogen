# Installation de programmes d'analyse de l'évolution moléculaire (des séquences)

PAML v1.3.1:  
```bash
cd /opt
sudo wget https://github.com/abacus-gene/paml/releases/download/4.10.7/paml-4.10.7-linux-X86_64.tgz
sudo tar -xzvf paml-4.10.7-linux-X86_64.tgz
sudo chown -R root:root paml-4.10.7
cd /opt/paml-4.10.7
sudo rm /opt/paml-4.10.7/bin/*
cd /opt/paml-4.10.7/src
sudo sed -i 's/CC = icc/CC = gcc/g' Makefile
sudo make -f Makefile
sudo mv baseml ../bin/
sudo mv basemlg ../bin/
sudo mv chi2 ../bin/
sudo mv codeml ../bin/
sudo mv evolver ../bin/
sudo mv infinitesites ../bin/
sudo mv mcmctree ../bin/
sudo mv pamp ../bin/
sudo mv yn00 ../bin/

## test ##
VERSION=4.10.7
mkdir -p ~/codeml_test

## copy required data
cd ~/codeml_test
cp /opt/paml-$VERSION/examples/codeml.ctl .
cp /opt/paml-$VERSION/examples/stewart.aa .
cp /opt/paml-$VERSION/examples/stewart.trees .

## modify control file to include path to AA rate file
sed -i "s/..\/dat/\/opt\/paml-$VERSION\/dat/g" codeml.ctl

## run codeml
/opt/paml-$VERSION/bin/codeml


```