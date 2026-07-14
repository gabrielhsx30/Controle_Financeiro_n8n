# Controle_Financeiro_n8n
Este workflow integra o Telegram ao Supabase usando n8n e IA (Gemini 2.0 Flash). Ele recebe mensagens de texto com despesas/receitas via bot, usa IA para estruturar os dados em JSON e os salva com segurança na tabela transacoes_financeiras do PostgreSQL/Supabase, enviando confirmação no Telegram.




# 🤖 Controle Financeiro Inteligente (Telegram + n8n + Supabase/PostgreSQL)

Este repositório contém o arquivo JSON de exportação de um workflow desenvolvido no **n8n**. Ele automatiza o registro de despesas e receitas enviadas por mensagens de texto em um bot do Telegram, processando a linguagem natural por meio de um agente de IA e persistindo os dados formatados diretamente no **Supabase** (via PostgreSQL).

---

## ⚙️ Como o Fluxo Funciona

```
[Telegram Bot] ──> [n8n Trigger] ──> [Filtro de Texto] ──> [Agente de IA (Gemini)]
                                                                   │
                                                                   ▼
[Confirmação no Telegram] <── [Update Supabase] <── [Sanitização (Node Code/JS)]

```

### Passo a Passo Sequencial:

1. **Entrada de Dados (`Telegram Trigger`)**: O bot recebe uma mensagem de áudio, imagem ou texto do usuário.
2. **Validação de Formato (`Is text message?`)**: O fluxo filtra mensagens que não sejam texto puro, respondendo ao usuário com uma instrução amigável caso envie mídias não suportadas.
3. **Processamento de Linguagem Natural (`Agente financeiro IA` + `Gemini`)**: A mensagem de texto é enviada para o modelo **Gemini 2.0 Flash**. Usando um *Structured Output Parser*, a IA identifica se é uma transação válida, classifica-a (categoria, valor, tipo de movimentação, forma de pagamento, data) e gera uma mensagem de retorno amigável.
4. **Validação da Transação (`É transação válida?`)**: Se o input não contiver informações mínimas (como valor ou contexto coerente), o fluxo é interrompido e uma mensagem de erro orientadora é devolvida ao usuário.
5. **Tratamento de Dados (`Transformar para PostgreSQL`)**: Um nó de código em JavaScript sanitiza os dados estruturados obtidos pela IA. Ele normaliza valores decimais, valida tipos (`despesa`/`receita`), formata datas e mapeia metadados do Telegram (ID do chat, ID da mensagem, username).
6. **Persistência (`Supabase - Update a row`)**: Atualiza ou insere o registro mapeado diretamente no banco de dados relacional do Supabase (tabela `transacoes_financeiras`).
7. **Feedback ao Usuário (`Confirmar registro no Telegram`)**: Após a garantia de gravação segura no banco, o bot retorna uma confirmação de sucesso para o chat do usuário.

---

## 🛠️ Tecnologias e Integrações Utilizadas

* **n8n**: Orquestração e automação do pipeline de dados.
* **Telegram Bot API**: Interface de interação com o usuário final.
* **Google Gemini (Gemini 2.0 Flash)**: Processamento de linguagem natural e extração estruturada de entidades em JSON.
* **Supabase / PostgreSQL**: Banco de dados relacional hospedado para armazenamento histórico seguro dos dados financeiros.
* **JavaScript (Node.js)**: Manipulação, formatação de moeda e validação de regras de negócio.

---

## 📋 Pré-requisitos para Implantação

### 1. Estrutura da Tabela no Supabase (PostgreSQL)

Certifique-se de criar a tabela `transacoes_financeiras` em seu schema público antes de ativar o fluxo:

```sql
CREATE TABLE public.transacoes_financeiras (
    id SERIAL PRIMARY KEY,
    data_transacao DATE NOT NULL,
    tipo VARCHAR(10) NOT NULL CHECK (tipo IN ('despesa', 'receita')),
    valor NUMERIC(10, 2) NOT NULL,
    moeda VARCHAR(3) DEFAULT 'BRL',
    categoria VARCHAR(50) DEFAULT 'Outros',
    descricao TEXT,
    forma_pagamento VARCHAR(30),
    observacao TEXT,
    telegram_chat_id VARCHAR(50),
    telegram_message_id VARCHAR(50) UNIQUE,
    telegram_username VARCHAR(100),
    mensagem_original TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);


