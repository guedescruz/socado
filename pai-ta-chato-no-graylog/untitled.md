# Configurando Graylog Master



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
{% tab title="Plain Text" %}
```text
apt install ntpdate
ntpdate a.pool.ntp.br
dpkg-reconfigure tzdata
```
{% endtab %}

{% tab title="Crontab - Update NTP" %}
```
#Update NTP
*/2 * * * * root /usr/sbin/ntpdate a.ntp.br

#Update OpenCTI Feeds
34 4 * * * root cd /opt/threatFeeds/ && git pull
```
{% endtab %}
{% endtabs %}

