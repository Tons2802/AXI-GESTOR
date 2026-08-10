# AXI-GESTOR — Automação de Pedidos e Produção (n8n)

Workflow n8n para gestão automática de pedidos de vendas, controle de estoque e geração de Ordens de Serviço (OS), integrado ao Google Sheets.

---

## 📋 O que faz

A cada **1 minuto**, o workflow:

1. Lê novos pedidos recebidos via **Google Forms**
2. Verifica o estoque disponível de cada produto e seus componentes
3. Gera automaticamente **Ordens de Serviço (OS)** de montagem e/ou reposição
4. Atualiza as quantidades de **estoque** (disponível e reservado)
5. Marca os pedidos como **processados**

---

## 🗂️ Planilhas Google Sheets utilizadas

O workflow utiliza **duas planilhas**:

### Planilha 1 — Formulário de Pedidos
| Aba | Descrição |
|---|---|
| `Respostas ao formulário 1` | Entrada de pedidos enviados pelo time de vendas via Google Forms |

### Planilha 2 — Ordem de Produção
| Aba | Descrição |
|---|---|
| `ESTOQUE` | Produtos com quantidade disponível, reservada e margem de segurança |
| `ESTRUTURA` | BOM (Bill of Materials) — estrutura de componentes de cada produto |
| `NIVEL 2` | Componentes de segundo nível (subprodutos com estrutura própria) |
| `OS` | Ordens de Serviço geradas (componentes, quantidades, operador, status) |
| `MONTAGEM` | Fila de montagem com situação de cada pedido |

---

## 🔄 Fluxo do Workflow

```
Schedule Trigger (1 min)
    │
    ▼
Busca pedidos NÃO processados (Formulário)
    │
    ▼
Filtra pedidos novos
    │
    ├── Carrega ESTRUTURA
    ├── Carrega ESTOQUE
    └── Carrega NIVEL 2
              │
              ▼
        Código JavaScript (análise de estoque)
              │
         ┌────┴────────────┐
         ▼                 ▼                 ▼
       _tipo: OS     _tipo: MONTAGEM    _tipo: ESTOQUE
         │                 │                 │
    Grava na aba OS   Verifica se        Atualiza qtd
                      precisa produção   disponível/reservada
                           │
                    PRODUÇÃO = SIM?
                           │
                    Gera OS de reposição
                           │
                           ▼
              Marca pedidos como PROCESSADOS
```

---

## ⚙️ Configuração

### Pré-requisitos

- **n8n** (self-hosted ou cloud)
- Conta de serviço **Google** com acesso às planilhas (Google Sheets API ativada)
- Duas planilhas Google Sheets configuradas conforme estrutura acima

### Passos para importar

1. No n8n, vá em **Workflows → Import**
2. Faça upload do arquivo `AXI-GESTOR.json`
3. Configure as credenciais:
   - Crie uma credencial do tipo **Google Sheets (Service Account)**
   - Substitua nos nós a referência de credencial pelo ID da sua conta de serviço
4. Atualize os IDs das planilhas em todos os nós do Google Sheets:
   - Substitua `SUA_PLANILHA_FORMULARIO_ID` pelo ID real da planilha de formulário
   - Substitua `SUA_PLANILHA_PRODUCAO_ID` pelo ID real da planilha de produção
5. Ative o workflow

### Estrutura esperada das planilhas

#### ESTOQUE
| PRODUTO | QTD DISPONIVEL | QTD RESERVADA | MARGEM DE SEGURANÇA |
|---|---|---|---|
| JAGUAR 10L | 50 | 0 | 10 |

#### ESTRUTURA
| Produto | Componente | Quantidade |
|---|---|---|
| JAGUAR 10L | CABO INOX | 2 |

#### NIVEL 2
| PRODUTO | COMPONENTE | QUANTIDADE |
|---|---|---|
| CESTO JAGUAR 10L BRANCO | JAGUAR 10L | 1 |

#### Respostas ao formulário 1
| Carimbo de data/hora | PRODUTO | QUANTIDADE | CLIENTE | Nº do Pedido | OBS | PROCESSADOS |
|---|---|---|---|---|---|---|

---

## ⚠️ Atenção antes de subir para o GitHub

Antes de versionar o arquivo `.json`, remova ou substitua por valores genéricos:

- **IDs das planilhas Google** (campo `value` nos nós Google Sheets)
- **ID da credencial** (campo `credentials.googleApi.id`)
- **Instance ID do n8n** (campo `meta.instanceId`)

Exemplo de substituição segura:
```json
"id": "SEU_CREDENTIAL_ID",
"instanceId": "SEU_INSTANCE_ID"
```

---

## 🏷️ Tecnologias

- [n8n](https://n8n.io/) — plataforma de automação
- Google Sheets API
- Google Forms
- JavaScript (nodes de código customizados)

---

## 📄 Licença

Este projeto é de uso interno. Adapte conforme necessário.
