Servidor Mosquitto com PSK
==========================

Guia completo de instalação e configuração do servidor Mosquitto MQTT usando Pre-Shared Keys (PSK) para autenticação

Introdução
----------

Este guia descreve o processo completo de instalação e configuração do servidor Mosquitto MQTT com autenticação baseada em Pre-Shared Keys (PSK). A configuração inclui dois listeners: um não seguro para testes e outro seguro usando PSK.

Instalação do Mosquitto
-----------------------

### 1\. Adicionar repositório oficial

Adicione o repositório oficial do Mosquitto no Ubuntu para garantir a instalação da versão mais recente.

### 2\. Atualizar lista de pacotes

Atualize a lista de pacotes do sistema após adicionar o novo repositório.

### 3\. Instalar Mosquitto e clientes

Instale o broker Mosquitto e as ferramentas de cliente (mosquitto\_pub e mosquitto\_sub) para testes.

*   **mosquitto** - broker MQTT
*   **mosquitto-clients** - ferramentas para publicação e assinatura de tópicos

### 4\. Verificar instalação

Verifique se a instalação foi bem-sucedida confirmando o caminho do executável.

Estrutura de Arquivos
---------------------

Todos os arquivos de configuração ficam localizados em:

/etc/mosquitto/

Arquivos importantes:

*   **mosquitto.conf** - configuração principal do broker
*   **psk.txt** - lista de PSKs (identity + chave)
*   **acl.conf** - regras de permissão para usuários

Configuração do PSK
-------------------

### 1\. Criar arquivo psk.txt

Crie o arquivo que armazenará as chaves PSK:

joao53:9f3b2d4c6a7e8b1f2233445566778899

Onde:

*   **joao53** - identity do cliente
*   **9f3b2d4c6a7e8b1f2233445566778899** - chave secreta (PSK)

### 2\. Configurar mosquitto.conf

Edite o arquivo de configuração principal:

per\_listener\_settings true  
  
\# Listener 1: sem segurança  
listener 1883  
allow\_anonymous true  
  
\# Listener 2: seguro com PSK  
listener 8883  
psk\_hint "hint-para-joao53"  
psk\_file /etc/mosquitto/psk.txt  
acl\_file /etc/mosquitto/acl.conf  
use\_identity\_as\_username true

Explicação das configurações:

*   **per\_listener\_settings true** - impede que configurações de um listener interfiram no outro
*   **listener 1883** - permite conexões não seguras (para testes)
*   **listener 8883** - porta segura usando PSK
*   **psk\_hint** - dica que o cliente vê ao se conectar
*   **psk\_file** - arquivo de chaves PSK
*   **acl\_file** - arquivo de regras de permissão
*   **use\_identity\_as\_username true** - Mosquitto usa a identity do PSK como nome de usuário

### 3\. Configurar ACL (acl.conf)

Defina as regras de permissão:

user joao53  
  
\# Permissões de escrita  
topic write joao53/pub/#  
topic write joao53/bitdoglab/#  
  
\# Permissões de leitura  
topic read joao53/pub/#  
topic read joao53/bitdoglab/#

Permite ao usuário **joao53** publicar e assinar qualquer subtópico de **joao53/pub/** e **joao53/bitdoglab/**.

**Nota:** O uso do caractere **#** como curinga permite operações em todos os subtópicos. Publicar exatamente em **joao53/pub** sem subtópico não é permitido com esta configuração.

Segurança dos Arquivos
----------------------

Proteja os arquivos sensíveis com as permissões adequadas:

\# Definir proprietário correto  
chown mosquitto:mosquitto /etc/mosquitto/acl.conf  
chown mosquitto:mosquitto /etc/mosquitto/psk.txt  
  
\# Definir permissões restritas  
chmod 0700 /etc/mosquitto/acl.conf  
chmod 0700 /etc/mosquitto/psk.txt

*   **chown** - define dono e grupo como mosquitto
*   **chmod 0700** - só o dono pode ler, escrever e executar

Inicialização do Mosquitto
--------------------------

### Reiniciar o serviço

Reinicie o serviço para aplicar as mudanças de configuração.

### Execução manual para depuração

Para ver logs detalhados e diagnosticar problemas, execute manualmente:

mosquitto -c /etc/mosquitto/mosquitto.conf -v

Os logs ajudam a diagnosticar problemas de conexão ou ACL.

Testes de Conexão
-----------------

### 1\. Subscriber (assinar tópicos)

Comando para assinar tópicos:

mosquitto\_sub -h localhost -p 8883 -t "joao53/pub/#" --psk-identity "joao53" --psk "9f3b2d4c6a7e8b1f2233445566778899" -d

Parâmetros:

*   **\-t "joao53/pub/#"** - recebe mensagens de subtópicos de joao53/pub/
*   **\-d** - modo debug para logs detalhados

### 2\. Publisher (enviar mensagens)

Comando para publicar mensagens:

mosquitto\_pub -h localhost -p 8883 -t "joao53/pub/teste1" -m "Mensagem de teste PSK" --psk-identity "joao53" --psk "9f3b2d4c6a7e8b1f2233445566778899"

A mensagem é publicada em um subtópico, porque o ACL só permite operações em **joao53/pub/#**.

O subscriber deve receber a mensagem instantaneamente após a publicação.

Guia criado para configuração de servidor Mosquitto com autenticação PSK
