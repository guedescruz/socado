# Vagrant

## Laboratório provisionado com Vagrant + VirtualBox

### Network

A variável inicial, define a minha placa de rede. E ao dizer ao vagrant que quero usar a placa em modo `bridge` eu chamo a minha variavel. \(ref -&gt; vagrant-lab/Vagrantfile:L.16\).

Se adiconar diretamente à linha que configura a placa, da na mesma groselha então, make your choice bro.

Por meio do Vagrant, automatizei a criação das minhas máquinas virtuais, onde o sistema operacional é o Centos 8 em \(ref -&gt; vagrant-lab/Vagrantfile:14\).

Sobre o endereçamento, eu apenas concatenei o meu prefixo de rede, com o endereço que quero que seja configurado no host.

Para configurações de DNS e gateway, usei o arquivo `auto.sh` \(red -&gt; vagrant-lab/auto.sh:3,4\), pois aparentemente, o vagrant só configura estaticamente, o IP, em `network-scripts`.

