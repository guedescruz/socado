# \#3 - Wazuh Single Node

## Instalando Wazuh Master

## Configurando Wazuh

### Configurando elasticsearch

{% code title="/etc/elasticsearch/jvm.options" %}
```text
-Xms<valor do Max size>g
-Xmx<valor do Max size>g
```
{% endcode %}

### Configurando Wazuh Single Node

### Configurando OpenVPN

{% tabs %}
{% tab title="\#1 Criando .conf" %}
{% code title="/usr/share/doc/openvpn/examples/sample-config-files/" %}
```
cat client.conf > /etc/openvpn/graylogVPNClient.conf
```
{% endcode %}
{% endtab %}

{% tab title="\#2 Chave TLS" %}
{% code title="/etc/openvpn/ta.key" %}
```
#Chave gerado no Graylog Master
#/etc/openvpn/ta.key
```
{% endcode %}

Esta chave é usada pelo arquivo configurado na etapa \#3.

\`\`
{% endtab %}

{% tab title="\#3 Configurando .conf" %}
{% code title="/etc/openvpn/graylogVPNClient.conf" %}
```
proto tcp
;proto udp
remote 209.126.85.1 25922
;remote myserver-1 1194

<ca>
#GERADO NO GRAYLOG MASTER
#/etc/CA-Graylog/pki/ca.crt
</ca>

<cert>
#GERADO NO GRAYLOG MASTER
#/etc/CA-Graylog/pki/issued/uranus.crt
</cert>

<key>
#GERADO NO GRAYLOG MASTER
#/etc/CA-Graylog/pki/private/uranus.key
</key>

cipher AES-128-CBC
txqueuelen 1000
ncp-disable
sndbuf 512000
rcvbuf 512000
fast-io
```
{% endcode %}

* **209.126.85.1** 👉 ****é o IP com quem queremos fechar a VPN;
* **25922**  👉 é a porta que deve estar aberta para esta comunicação no outro servidor. \(Verifica com`netstat -lnp`\);
* Dentro das tags `<ca></ca>`, `<cert></cert>`e`<key></key>`deve ser inserido o conteudo do gerado duante a[ etapa \#1 de configuração da VPN no Graylog](bruno.md#configurando-o-openvpn);
* Também mude o algorito de criptografia de `AES-256` para `AES-128`;

### Configurando Elasticsearch
{% endtab %}

{% tab title="\#4 UFW" %}
```
ufw allow 25922
```
{% endtab %}

{% tab title="\#4 Reiniciando Serviço" %}
```
openvpn graylogVPNClient.conf
systemctl start openvpn@graylogVPNClient
systemctl enable openvpn@graylogVPNClient
systemctl restart openvpn@graylogVPNClient
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="\#1" %}
{% code title="/etc/elasticsearch/elaticsearch.yml" %}
```text
network.host: 127.0.0.1
node.name: node-1
cluster.initial_master_nodes:
        - node-1
```
{% endcode %}
{% endtab %}

{% tab title="\#2 Reiniciando o Serviço" %}
```
systemctl stop elasticsearch.service
systemctl start elasticsearch.service
```
{% endtab %}

{% tab title="\#3 Verificando" %}
```
curl -u "<user>:<passwd>" -k https://127.0.0.1:9200/_cluster/health?pretty
```
{% endtab %}
{% endtabs %}



### Configurando Nginx no Wazuh

```text
apt install nginx
```

### Configurando envio de logs ao Graylog

{% tabs %}
{% tab title="\#1 Remote" %}
Use o comando `echo $RANDOM` e use sua saída como porta para os clients Wazuh realizerem o envio dos dados coletados.

{% code title="/var/ossec/etc/ossec.conf" %}
```text
  <remote>
    <port>12572</port>
  </remote>
```
{% endcode %}
{% endtab %}

{% tab title="\#2 Auth" %}
Use o comando `echo $RANDOM` e use sua saída como porta para os clients Wazuh se autenticarem.

{% code title="/var/ossec/etc/ossec.conf" %}
```
  <auth>
    <port>14516</port>
    <use_password>yes</use_password>
    <ssl_auto_negotiate>yes</ssl_auto_negotiate>
  </auth>
```
{% endcode %}
{% endtab %}

{% tab title="\#3 Syslog Output" %}
{% code title="/var/ossec/etc/ossec.conf" %}
```text
  <syslog_output>
    <server>172.27.222.1</server>
    <port>30000</port>
    <format>json</format>
  </syslog_output>
```
{% endcode %}
{% endtab %}

{% tab title="\#4 Portas" %}
Execute o comando abaixo no servidor Graylog Master

```text
ufw allow from 172.27.222.0/24 to 172.27.222.1 port 30000 proto udp
```

Liberando a comunicação com a porta 30000 entre os servidores no firewall.

Feito isso, volte ao Wazuh Master e reinicie o serviço com:

```text
/etc/init.d/wazuh-manager restart
```

A partir deste ponto, já deve ser possível visualizar os logs no Graylog Master.
{% endtab %}
{% endtabs %}





