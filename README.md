# Cash Out via Pix - Workflow Automatizado com AWS Step Functions

Este repositório demonstra um workflow automatizado para **solicitações de cash out via Pix** em uma fintech, utilizando **AWS Step Functions**, **AWS Lambda** e **DynamoDB**.

---

## Visão Geral

O fluxo automatiza a retirada de saldo via Pix, passando por validação, checagem de saldo, execução do Pix, atualização de saldo, registro da transação e notificação ao cliente.

### Fluxo Principal

1. **Receber solicitação** → API ou evento SNS/SQS inicia o workflow.
2. **Validar dados** → Verifica chave Pix, valor e formato.
3. **Checar saldo** → Confere se o cliente tem saldo disponível.
4. **Executar Pix** → Envia o comando ao gateway Pix (SPI/PSP).
5. **Atualizar saldo** → Deduz valor da conta interna.
6. **Registrar transação** → Cria log e registro de auditoria.
7. **Notificar cliente** → Envia e-mail, push ou webhook com o status da operação.

---

## Diagrama do Workflow


<img width="918" height="969" alt="stepfunctions_graph" src="https://github.com/user-attachments/assets/b4ef44d8-e778-4777-835d-95de1cb77823" />

---

## Arquitetura AWS

| Serviço | Função |
|----------|--------|
| **AWS Step Functions** | Orquestra o fluxo de cash out. |
| **AWS Lambda** | Executa a lógica de cada etapa. |
| **Amazon DynamoDB** | Armazena dados de saldo e logs de transações. |
| **Amazon CloudWatch** | Monitora execuções e erros. |
| **Amazon SNS** | Envia notificações de sucesso ou falha. |


## Logs e Monitoramento

- Cada execução do Step Function é registrada no AWS CloudWatch.
- As falhas disparam alertas via SNS para o time de operações.
