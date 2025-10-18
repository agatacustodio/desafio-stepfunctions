# Cash Out via Pix - Workflow Automatizado com AWS Step Functions

Este repositório demonstra um workflow automatizado para **solicitações de cash out via Pix** em uma fintech, utilizando **AWS Step Functions**, **AWS Lambda** e **DynamoDB**.

---

## O que é o AWS Step Functions?

É um serviço de orquestração sem servidor que ajuda a construir aplicativos distribuídos, automatizar processos e orquestrar microsserviços conectando-os em fluxos de trabalho visuais.

---

## Visão Geral

O fluxo automatiza a retirada de saldo via Pix. Ele passa por validação, checagem de saldo e, se aprovado, **deduz o valor e tenta executar o Pix**. O fluxo contém lógica robusta para **tratamento de falhas**, incluindo estorno do saldo em caso de erro na API Pix ou registro de falha em qualquer etapa.

## Fluxo Principal

1.  **Receber solicitação**: API ou evento SNS/SQS inicia o workflow.
2.  **Validar dados** (`ValidarSolicitacao`): Verifica chave Pix, valor e formato.
3.  **Checar saldo** (`ChecarSaldo`): Confere se o cliente tem saldo disponível.
    * *Caso não tenha saldo, segue para o **Registro de Falha**.*
4.  **Deduzir saldo** (`DeduzirSaldo`): **Deduz** o valor da conta interna. (Etapa prévia à execução do Pix).
5.  **Executar Pix** (`ExecutarPix`): Envia o comando ao gateway Pix (SPI/PSP).
6.  **Tratamento de Erro Pix** (`ErroNaAPIPix?`): Se houver erro de comunicação com o gateway, o fluxo segue para:
    * **Estornar Saldo** (`EstornarSaldo`) e, em seguida, **Notificar Cliente**.
7.  **Notificar cliente** (`NotificarCliente`): Envia e-mail, push ou webhook com o status da operação (sucesso ou falha).
8.  **Registrar transação** (`RegistrarTransacao`): Cria log e registro de auditoria, marcando a transação como Sucesso.

## Tratamento de Falhas

O diagrama prevê o **Registro de Falha** (`RegistrarFalha`) em diversas situações, como:
* Falha na `ValidarSolicitacao`, `ChecarSaldo` ou `DeduzirSaldo`.
* Cliente sem saldo suficiente (`TemSaldo?` -> `Default`).
* Falha na comunicação com o `ExecutarPix`.

Em todos os casos de falha (exceto falha na API Pix que segue estorno), o fluxo passa pela etapa **`RegistrarFalha`** antes de seguir para **`RegistrarTransacao`** (marcando a transação como Falha).

## Diagrama do Workflow

<img width="918" height="969" alt="stepfunctions_graph" src="https://github.com/user-attachments/assets/5476f615-5677-4efb-8478-f53fe8320ed3" />

## Arquitetura AWS

| Serviço | Função |
| :--- | :--- |
| **AWS Step Functions** | Orquestra o fluxo de cash out. |
| **AWS Lambda** | Executa a lógica de cada etapa (ex: validação, checagem, dedução, Pix, estorno, notificação). |
| **Amazon DynamoDB** | Armazena dados de saldo e logs de transações (interação com `ChecarSaldo`, `DeduzirSaldo` e `RegistrarTransacao`). |
| **Amazon CloudWatch** | Monitora execuções e erros. |
| **Amazon SNS** | Utilizado primariamente pela Lambda `NotificarCliente` para enviar comunicações (e-mail, push) e pelo CloudWatch para alertas de falha. |

## Logs e Monitoramento

Cada execução do Step Function é registrada no AWS CloudWatch.
As falhas críticas (erros de execução) disparam alertas via SNS para o time de operações.
