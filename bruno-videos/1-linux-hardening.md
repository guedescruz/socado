# \#1 - Linux Hardening

## SO Hardening

### Atualizando repositório, pacotes e instalando ferramentas

{% tabs %}
{% tab title="\#1 Update" %}
```text
apt-get update && apt-get dist-upgrade
```
{% endtab %}

{% tab title="\#2 Install" %}
```
apt install htop pmisc tcpdump iptraf sysstat zfsutils-linux fail2ban openvpn easy-rsa
```
{% endtab %}

{% tab title="\#3 Install" %}
```text
apt install zfsutils-linux fail2ban openvpn easy-rsa
```
{% endtab %}
{% endtabs %}

### Implementando ZFS

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
{% tab title="\#4 FSTAB Bombado" %}
{% code title="/etc/fstab" %}
```bash
UUID=<UUID> /        ext4    errors=remount-ro,noatime,nodiratim    0 0
UUID=<UUID> /boot    ext4    defaults,noatime,nodiratime            0 0
/swapfile   none     swap    sw                                     0 0
```
{% endcode %}
{% endtab %}

{% tab title="\#4 Carregando as configs" %}
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
{% tab title="\#6 Diretorio" %}
```text
mkdir -p /opt/.fakedisk
nohup dd status=progress if=/dev/zero oflag=direct of=fakedisk1 bs=1MB count=500000 & 
```

É necessário criar um fakedisk pois, o ZFS é feito em cima de um disco.
{% endtab %}

{% tab title="\#7 Verificando" %}
```
iostat -m 1
ls -l
df -h
```
{% endtab %}
{% endtabs %}



### Network Hardening

{% tabs %}
{% tab title="Bloqueando portas" %}
```bash
ufw allow from 127.0.0.1
ufw allow 5022/tcp
ufw allow from https
ufw enable
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





