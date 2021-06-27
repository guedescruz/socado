# \#4 - Coleta no Graylog

Para receber os logs, é necessário criar um **Input** no Graylog, para isso, entre na interface e vá em System / Inputs clique em Select Input, selecione o formato Syslog UDP, e agora clique em Launch Input. Preencha os campos da seguinte forma:



![](../.gitbook/assets/image%20%285%29.png)

{% hint style="info" %}
Lembre-se de substituir a porta, pela que foi configurada no output do Wazuh
{% endhint %}

Depois de criado este input, clique em **More Actions**, depois em **Add static field**:

![](../.gitbook/assets/image%20%282%29.png)

### Criando um Indice

Clique em **System &gt;&gt; Indices &gt;&gt; Create Index** e altere apenas, e customize as seguintes opções:

{% hint style="info" %}
Relembrando, essas opções podem variar de acordo com o objetivo da coleta
{% endhint %}

1. **Select retention strategy** 👉 Delete Index
2. **Number of indices**👉93

