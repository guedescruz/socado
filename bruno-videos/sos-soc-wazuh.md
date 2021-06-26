# \#3 - Wazuh Single Node

## SO Hardening 

### Pacotes Atualizados

{% tabs %}
{% tab title="Atualização do SO" %}
```
apt update && apt dist-upgrade
```
{% endtab %}

{% tab title="Instalação de Pacotes" %}
```
apt install htop pmisc tcpdump iptraf openvpn easy-rsa sysstat fail2ban ethtool zfsutils-linux
```
{% endtab %}
{% endtabs %}

### SWAP

{% tabs %}
{% tab title="Verificando meomria SWAP" %}
```bash
free -m
```
{% endtab %}

{% tab title="Verificando swapiness" %}
```bash
sysctl -a | grep swap
```
{% endtab %}

{% tab title="sysctl config" %}
{% code title="/etc/sysctl.conf" %}
```bash
vim.swapiness = 01
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="\#1 - Ativando o módulo" %}
```text
modprobe zfs
```
{% endtab %}

{% tab title="\#2 - Adicionado ao arquivo" %}
{% code title="/etc/modules" %}
```
zfs
```
{% endcode %}
{% endtab %}

{% tab title="\#3 - Init Frames" %}
```
update-initramfs -k all -u
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="FSTAB Bombado" %}
{% code title="/etc/fstab" %}
```bash
UUID=<UUID> /        ext4    errors=remount-ro,noatime,nodiratim    0 0
UUID=<UUID> /boot    ext4    defaults,noatime,nodiratime            0 0
/swapfile   none     swap    sw                                     0 0
```
{% endcode %}
{% endtab %}

{% tab title="Carregando as configs" %}
```
mount -o remount,noatime,nodiratime /
mount -o remount,noatime,nodiratime /boot
tune2fs -o journal_data_writeback /dev/sda1
tune2fs -o journal_data_writeback /dev/sda2
mount
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Diretorio" %}
```text
mkdir -p /opt/.fakedisk
```

É necessário criar um fakedisk pois, o ZFS é feito em cima de um disco.
{% endtab %}

{% tab title="DD" %}
```
nohup dd status=progress if=/dev/zero oflag=direct of=fakedisk1 bs=1MB count=5000000 & 
```



{% hint style="info" %}
Max size \(high water\):
{% endhint %}
{% endtab %}

{% tab title="Verificando" %}
```
iostat -m 1
ls -l
df -h
```
{% endtab %}
{% endtabs %}

### Instalação NTP

{% tabs %}
{% tab title="Configurando NTP" %}
```text
apt install ntpdate
ntpdate a.ntp.br
dpkg-reconfigure tzdata
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Crontab - Update NTP" %}
```
#Update NTP
*/2 * * * * root /usr/sbin/ntpdate a.ntp.br

#Update OpenCTI Feeds
34 4 * * * root cd /opt/threatFeeds/ && git pull
```
{% endtab %}
{% endtabs %}

### Network Hardening

{% tabs %}
{% tab title="Bloqueando portas" %}
```bash
ufw default deny incoming
ufw allow from 127.0.0.1
ufw allow 5022/tcp
ufw allow from https
```
{% endtab %}

{% tab title="SSH" %}
{% code title="/etc/ssh/sshd\_config" %}
```
Port 5022
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Fail2Ban - IP Blocklist

{% tabs %}
{% tab title="Configurando" %}
{% code title="/etc/fail2ban/jail.d/default-debian.conf" %}
```text
[sshd]
enabled = true
port = 5022
```
{% endcode %}
{% endtab %}

{% tab title="Reiniciando o Serviço" %}
```
/etc/init.d/fail2ban restart
```
{% endtab %}

{% tab title="Verificando o arquivo" %}
```
fail2ban-client status sshd
```
{% endtab %}
{% endtabs %}

### Configurando elasticsearch

{% code title="/etc/elasticsearch/jvm.options" %}
```text
-Xms<valor do Max size>g
-Xmx<valor do Max size>g
```
{% endcode %}

### Configurando Wazuh Single Node

{% tabs %}
{% tab title="\#1 Alterando Network " %}
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

### Configurando OpenVPN

{% tabs %}
{% tab title="\#1 Criando .conf" %}
{% code title="/usr/share/doc/openvpn/examples/sample-config-files/" %}
```
cat client.conf > /etc/openvpn/graylogVPNClient.conf
```
{% endcode %}
{% endtab %}

{% tab title="\#2 Criando chave tls" %}
{% code title="/etc/openvpn/ta.key" %}
```
#Chave gerado no Graylog Master
#Cole aqui
```
{% endcode %}

* Essa chave foi gerada em 
{% endtab %}

{% tab title="\#3 Configurando .conf" %}
{% code title="/etc/openvpn/graylogVPNClient.conf" %}
```
proto tcp
;proto udp
remote 209.126.85.1 25922
;remote myserver-1 1194
<ca>
#CONTEUDO DO CERTIFICADO 
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

tls-auth ta.key 1
cipher AES-128-CBC

```
{% endcode %}

* **209.126.85.1** 👉 ****é o IP com quem queremos fechar a VPN;
* **25922**  👉 é a porta que deve estar aberta para esta comunicação no outro servidor. \(Verifica com`netstat -lnp`\);
* Dentro das tags `<ca></ca>`, `<cert></cert>`e`<key></key>`deve ser inserido o conteudo do gerado duante a[ etapa \#1 de configuração da VPN no Graylog](bruno.md#configurando-o-openvpn);
* Também mude o algorito de criptografia de `AES-256` para `AES-128`;
* 
{% endtab %}

{% tab title="\#4 Reiniciando Serviço" %}
```
systemctl start openvpn@graylogVPNClient
systemctl enable openvpn@graylogVPNClient
systemctl restart openvpn@graylogVPNClient
```
{% endtab %}
{% endtabs %}









### Configurando Nginx no Wazuh

```text
apt install nginx
```

