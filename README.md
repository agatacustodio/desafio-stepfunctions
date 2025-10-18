
# Explorando Workflows Automatizados com AWS Step Functions

Este laboratório tem como objetivo consolidar meus conhecimentos em workflows automatizados com AWS Step Functions. O caso de uso que escolhi se relaciona com o meu background profissional.


# 💸 Cash Out via Pix - Workflow Automatizado com AWS Step Functions

Este repositório demonstra um **workflow automatizado** para **solicitações de cash out via Pix** em uma fintech, utilizando **AWS Step Functions**, **AWS Lambda** e **DynamoDB**.

---

## 🏗️ Visão Geral

O processo automatiza a retirada de saldo de clientes via **Pix**, garantindo segurança, rastreabilidade e integração com o **gateway de pagamentos**.

### 🔄 Fluxo Principal

1. **Receber solicitação** → API ou evento SNS/SQS inicia o workflow.
2. **Validar dados** → Verifica chave Pix, valor e formato.
3. **Checar saldo** → Confere se o cliente tem saldo disponível.
4. **Executar Pix** → Envia o comando ao gateway Pix (SPI/PSP).
5. **Atualizar saldo** → Deduz valor da conta interna.
6. **Registrar transação** → Log e auditoria.
7. **Notificar cliente** → Envia e-mail, push ou webhook de status.

---

## 🧩 Diagrama do Workflow

![Workflow Cash Out via Pix](docs/cashout-diagram.png)

---

## 🧠 Arquitetura AWS

| Serviço | Função |
|----------|--------|
| **Step Functions** | Orquestra o fluxo de cash out. |
| **AWS Lambda** | Executa a lógica de cada etapa. |
| **DynamoDB** | Armazena dados de saldo e logs de transações. |
| **CloudWatch** | Monitora execuções e erros. |
| **SNS/SES** | Notificação de sucesso ou falha. |

---

## 📜 Definição do Workflow (Amazon States Language)

Arquivo: `stepfunctions/cashout-pix.asl.json`

```json
{
  "Comment": "Workflow de Cash Out via Pix",
  "StartAt": "ValidarSolicitacao",
  "States": {
    "ValidarSolicitacao": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:validarSolicitacaoPix",
      "Next": "ChecarSaldo"
    },
    "ChecarSaldo": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:checarSaldoCliente",
      "Next": "TemSaldo?",
      "Catch": [{ "ErrorEquals": ["SaldoInsuficienteError"], "Next": "RegistrarFalha" }]
    },
    "TemSaldo?": {
      "Type": "Choice",
      "Choices": [{ "Variable": "$.saldoSuficiente", "BooleanEquals": true, "Next": "ExecutarPix" }],
      "Default": "RegistrarFalha"
    },
    "ExecutarPix": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:executarPix",
      "Next": "AtualizarSaldo",
      "Catch": [{ "ErrorEquals": ["PixGatewayError"], "Next": "RegistrarFalha" }]
    },
    "AtualizarSaldo": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:atualizarSaldo",
      "Next": "RegistrarTransacao"
    },
    "RegistrarTransacao": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:registrarTransacao",
      "Next": "NotificarCliente"
    },
    "RegistrarFalha": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:registrarFalha",
      "Next": "NotificarCliente"
    },
    "NotificarCliente": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:::function:notificarCliente",
      "End": true
    }
  }
}
```

## ⚙️ Lambdas Incluídas

Cada função Lambda está dentro da pasta lambdas/ e possui um index.js de exemplo.

Exemplo: lambdas/executarPix/index.js

exports.handler = async (event) => {
  console.log("Executando Pix...", event);
  const { valor, chavePix } = event;

  try {
    // Simulação de chamada ao gateway Pix
    const response = { status: "SUCESSO", txid: "TX123ABC" };
    return { ...response, sucesso: true };
  } catch (error) {
    throw new Error("PixGatewayError");
  }
};

## 🚀 Implantação com AWS SAM

sam build

sam deploy --guided


## 🧾 Logs e Monitoramento

Cada execução do Step Function é registrada no AWS CloudWatch.

As falhas disparam alertas via SNS para o time de operações.

## 🔒 Segurança

Permissões IAM mínimas necessárias (least privilege).

Tokens e chaves Pix armazenadas no AWS Secrets Manager.

Auditoria total de transações com CloudTrail.

## 🧰 Tecnologias

AWS Step Functions

AWS Lambda (Node.js 18.x)

DynamoDB

Amazon SNS / SES

AWS SAM
