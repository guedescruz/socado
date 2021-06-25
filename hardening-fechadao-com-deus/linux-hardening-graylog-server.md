# Linux Hardening - Graylog Server

## Hardening pão com ovo

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

{% hint style="info" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

UFW

```bash
ufw allow from 127.0.0.1
ufw allow 5022/tcp
ufw allow from https
```

SSH

{% code title="/etc/ssh/sshd\_config" %}
```bash
Port 5022
```
{% endcode %}

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

Verificando

{% tabs %}
{% tab title="Portas abertas" %}
```text
netstat -lnp
```
{% endtab %}

{% tab title="Logs de acesso" %}
```

```
{% endtab %}
{% endtabs %}

## Hardening/Tunning Graylog

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

Ponto de montagem, roda um `df -Th`pra ver o antes e o depois.

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
mkdir -p /opt/fakedisk

```
{% endtab %}

{% tab title="DD" %}
```
dd status=progress if=/dev/zero of=/fakedisk1 bs=32MB count=46875
```
{% endtab %}

{% tab title="Verificando" %}
```
ls -l
df -h
```
{% endtab %}
{% endtabs %}

