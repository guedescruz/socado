# \#2 - Graylog Single Node

### SO Hardening

Becoming a super hero is a fairly straight forward process:

{% tabs %}
{% tab title="Atualização do SO" %}
```
apt update && apt dist-upgrade
```
{% endtab %}

{% tab title="Instalação de Pacotes" %}
```
apt install htop pmisc tcpdump iptraf sysstat zfsutils-linux fail2ban
```
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

```text
apt-cache search iptables auto deny
apt-cache search iptables auto block
apt-cache search quarentine auto
apt-cache search quarentine
```

### Network Hardening

{% tabs %}
{% tab title="Bloqueando portas" %}
```bash
ufw allow from 127.0.0.1
ufw allow 5022/tcp
ufw allow from https
```
{% endtab %}

{% tab title="SSH Server" %}
{% code title="/etc/ssh/sshd\_config" %}
```
Port 5022
```
{% endcode %}
{% endtab %}
{% endtabs %}

### Fail2Ban - IP Blocklist

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

### ZFS

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

{% tab title="" %}
{% code title="/etc/sysctl.conf" %}
```bash
vim.swapiness = 01
```
{% endcode %}
{% endtab %}
{% endtabs %}

Verifique os pontos de montagem com `df -Th`pra ver o antes e o depois.

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
nohup dd status=progress if=/dev/zero oflag=direct of=fakedisk1 bs=1MB count=500000 & 
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

Observe o valor `Max size (high water):`

{% tabs %}
{% tab title="Verificando Max size" %}
```text
arc_summary -a
```
{% endtab %}

{% tab title="Java ES" %}
{% code title="/etc/elasticsearch/jvm.options" %}
```
-Xms<valor do Max size>g
-Xmx<valor do Max size>g
```
{% endcode %}
{% endtab %}

{% tab title="Java Graylog" %}
{% code title="/etc/default/graylog-server" %}
```
GRAYLOG_SERVER_JAVA_OPTS="-Xms<Metade do MAX>g -Xmx<Metade do MAX>g (...)"
```
{% endcode %}
{% endtab %}

{% tab title="Reiniciando o serviço" %}
```

systemctl restart elasticsearch.service graylog-server.service
```
{% endtab %}
{% endtabs %}

Configurando Elasticsearch Master e Graylog Master

{% tabs %}
{% tab title="Elasticsearch Master" %}
{% code title="/etc/elasticsearch/elasticsearch.yml" %}
```text
cluster.name: graylog
node.name: node-1
network.host: 209.126.85.1
http.port: 9200
action.auto_create_index: false
http.max_initial_line_length: 16KB
http.max_content_length: 300mb
```
{% endcode %}
{% endtab %}

{% tab title="Graylog Master" %}
{% code title="/etc/graylog/server/server.conf" %}
```
http_bind_address = 209.126.85.1:9000
elasticsearch_hosts = http://209.126.85.1:9200
root_timezone = America/Sao_Paulo
output_batch_size = 1000
processbuffer_processors = 6
outputbuffer_processors =  6
outputbuffer_processor_threads_max_pool_size = 128
elasticsearch_max_total_connections = 128
elasticsearch_max_total_connections_per_route = 128
```
{% endcode %}

Essas propriedades já existem no arquivo, então, pra não dar bigode, comente as linhas que correspondem que já existiam, e mantenha as novas descomentadas.
{% endtab %}

{% tab title="Verificando Cluster" %}
```text
curl 209.126.85.1:9200/_cluster/status
curl 209.126.85.1:9200/_cluster/health?pretty
```
{% endtab %}

{% tab title="Reiniciar serviços" %}
```

```
{% endtab %}
{% endtabs %}

Configurando MongoDB

{% tabs %}
{% tab title="MongoDB" %}
{% code title="/etc/mongodb.conf" %}
```text
bind_ip = 209.126.85.1
```
{% endcode %}
{% endtab %}

{% tab title="Graylog" %}
{% code title="/etc/graylog/server/server.conf" %}
```
mongodb_uri = mongodb://209.126.85.1/graylog
```
{% endcode %}
{% endtab %}

{% tab title="" %}
```

```
{% endtab %}
{% endtabs %}

### Configuração do NGINX

{% tabs %}
{% tab title="Install" %}
```text
apt install nginx
rm /etc/nginx/sites-available/default
```
{% endtab %}

{% tab title="Nginx - Graylog" %}
{% code title="/etc/nginx/sites-enabled/graylog" %}
```
server
{
    listen      443 ssl http2;
    server_name 209.126.85.1;
    ssl_certificate     /etc/nginx/certs/next4sec.com/next4sec-cert.pem;
    ssl_certificate_key /etc/nginx/certs/next4sec.com/next4sec-privkey.pem;
    ssl_protocols       TLSv1.1 TLSv1.2;
    # <- your SSL Settings here!

    location /
    {
      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Server $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Graylog-Server-URL https://$server_name/;
      proxy_pass       http://209.126.85.1:9000;
    }
}
```
{% endcode %}

Será necessário copiar os certificados para as pastas correspondentes
{% endtab %}
{% endtabs %}

### Instalação NTP

{% tabs %}
{% tab title="Configurando NTP" %}
```text
apt install ntpdate
ntpdate a.pool.ntp.br
```

Configure o dpkg

```text
dpkg-reconfigure tzdata
```

### Configurando OpenVPN
{% endtab %}

{% tab title="Crontab - Update NTP" %}
{% code title="/etc/crontab" %}
```
#Update NTP
*/2 * * * * root /usr/sbin/ntpdate a.ntp.br

#Update OpenCTI Feeds
34 4 * * * root cd /opt/threatFeeds/ && git pull
```
{% endcode %}

### Instalação do OpenVPN
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="\#1 Install" %}
```text
apt install openvpn easy-rsa
```
{% endtab %}

{% tab title="\#2 Gerando" %}
```
/usr/bin/make-cadir /etc/CA-Graylog/
cd /etc/CA-Graylog
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-dh
./easyrsa gen-crl
./easyrsa build-server-full graylog-server nopass
cd pki && ls
cd issued && ls
cd ../private && ls
```
{% endtab %}
{% endtabs %}

```text
cd /etc/graylog/
mkdir lookups/
wget --no-check-certificate -O file.gz "<lookuptables>"
gunzip file.gz
mv file GeoLite2-City.mmdb
```

### Message Processors Configuration

Vá na interface gráfica em **System/Configurations,** onde a ordem de **Message Processor Configuration** ficará a seguinte:

![](../.gitbook/assets/image.png)

Em **Geo-Location Processor**, clique em **Update**, e insira o caminho do diretório anteriormente criado `/etc/graylog/lookups/GeoLite2-City.mmdb`

![](../.gitbook/assets/image%20%282%29.png)

Voltando ao Graylog, no arquivo de configuração, mude a seguinte linha:

```text
allow_leading_wildcard_searches = true
```



