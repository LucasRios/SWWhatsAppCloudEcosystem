# SW WhatsApp Cloud Ecosystem 🚀

Este ecossistema é uma solução de mensageria corporativa escalável para WhatsApp, integrada à **API Oficial (Meta)** e **Whapi**. A arquitetura foi desenhada seguindo os princípios de **Microserviços Serverless**, utilizando AWS Lambda, SQS e SQL Server para suportar múltiplos clientes (Multi-tenant).



---

## 🏗️ Arquitetura e Fluxos

O sistema é dividido em dois grandes eixos operacionais que garantem alta disponibilidade e resiliência:

### 📥 1. Fluxo de Recebimento (Inbound)
Responsável por capturar, processar e armazenar mensagens vindas dos clientes.
* **[SQS Receiver](https://github.com/LucasRios/SWWhatsAppLambdaSQSReceiver)**: Webhook principal. Recebe o JSON da Meta/Whapi, processa mídias (upload para S3) e organiza o payload para processamento assíncrono.
* **[SQL Writer](https://github.com/LucasRios/SWWhatsAppLambdaSQLWriter)**: O roteador multi-tenant. Identifica o banco de dados de destino de cada cliente e persiste a mensagem no tenant correto.

### 📤 2. Fluxo de Envio (Outbound)
Responsável por orquestrar e disparar mensagens originadas pelo sistema.
* **[Collector](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvio)**: Realiza o polling nos bancos de dados dos clientes, reserva mensagens pendentes e as injeta na fila SQS FIFO.
* **[PostAPI (Executor)](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvioPostAPI)**: O motor de envio. Trata requisitos de mídia (como o fluxo de upload da Chakra), injeta credenciais e faz o disparo para as APIs.
* **[Salva Retorno](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvioSalvaRetorno)**: Fecha o ciclo de vida da mensagem, atualizando o status final (sucesso/erro) no banco do cliente.

### 🔐 3. Segurança e Identidade
* **[Get Meta Credentials](https://github.com/LucasRios/SWWhatsAppLambdaGetMetaCredentials)**: Broker de segurança que centraliza a recuperação de tokens e mapeamento de bases de dados, isolando informações sensíveis.

---

## 🔗 Repositórios do Ecossistema

| Repositório | Função | Tecnologia |
| :--- | :--- | :--- |
| [Receiver](https://github.com/LucasRios/SWWhatsAppLambdaSQSReceiver) | Webhook e Handler de Mídia (S3) | .NET / AWS S3 |
| [SQL Writer](https://github.com/LucasRios/SWWhatsAppLambdaSQLWriter) | Persistência Multi-tenant | .NET / SQL Server |
| [Collector](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvio) | Orquestrador de Mensagens Pendentes | .NET / SQS FIFO |
| [PostAPI](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvioPostAPI) | Executor de Envios e Integração Chakra | .NET / HttpClient |
| [Salva Retorno](https://github.com/LucasRios/SWWhatsAppLambdaSQSEnvioSalvaRetorno) | Callback de Status de Envio | .NET / SQL Server |
| [Credentials](https://github.com/LucasRios/SWWhatsAppLambdaGetMetaCredentials) | Broker de Tokens e Conexões | .NET / Security |

---

## 🛠️ Stack Tecnológica

* **Runtime:** .NET 6/8 (AWS Lambda)
* **Mensageria:** Amazon SQS (Standard & FIFO)
* **Storage:** Amazon S3 (Mídias e Documentos)
* **Banco de Dados:** Microsoft SQL Server (Relacional/Multi-tenant)
* **Logging:** AWS CloudWatch

## 🚀 Diferenciais da Solução

* **Escalabilidade Infinita:** Processamento distribuído que suporta grandes volumes de mensagens sem concorrência de banco de dados.
* **Resiliência (DLQ):** Falhas em APIs externas não perdem dados; as mensagens retornam para filas de reprocessamento.
* **Multi-tenancy:** Arquitetura preparada para isolamento de dados por banco de dados, facilitando a conformidade com a LGPD.

---
**Desenvolvido por [Lucas Rios](https://github.com/LucasRios)**
