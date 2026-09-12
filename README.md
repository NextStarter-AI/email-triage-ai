# Manual de Instalação — Triagem e Resposta Automática de E-mails com IA

## O que esse template faz

Esse fluxo de trabalho lê automaticamente os e-mails recebidos na sua caixa de entrada, usa Inteligência Artificial para entender do que cada um se trata (Suporte, Vendas, Financeiro ou Outro) e decide o que fazer em seguida:

- Se for uma pergunta simples de suporte e a IA tiver alta confiança na resposta → ela **responde automaticamente**.
- Se for uma pergunta de suporte mas a IA não tiver confiança suficiente → ela **notifica sua equipe no Slack** para revisão manual.
- Se for Vendas ou Financeiro → ela **notifica o canal certo no Slack** com um resumo do e-mail.
- Se não conseguir classificar o e-mail → ele cai em uma categoria de segurança, notificando a equipe geral.

Isso reduz o tempo de resposta e evita que e-mails importantes fiquem parados sem resposta na caixa de entrada.

---

## Passo 1 — Preparar o Gmail

1. Crie dois rótulos (labels) no Gmail: **"AI-Replied"** e **"Needs-Review"**.
2. Certifique-se de que a conta conectada tenha permissões de leitura e envio (isso é padrão na autenticação OAuth do Gmail).

## Passo 2 — Preparar o Slack

1. Crie (ou use canais já existentes) para receber as notificações — por exemplo: `#support`, `#sales`, `#finance`, `#general`.
2. Adicione o bot do Slack (que você vai conectar no Passo 4) a esses canais.

---

## Passo 3 — Importar o fluxo de trabalho no n8n

1. No n8n, clique em **"+ Add workflow"**.
2. Clique nos três pontinhos (**⋯**) → **"Import from File"**.
3. Selecione o arquivo `email-triage-ai-en.json`.

---

## Passo 4 — Conectar as credenciais

**Gmail:**
1. Dê um duplo clique no node **"New Email Received"**.
2. Em Credential → "Create New" → faça login com a conta do Gmail que você quer monitorar.
3. Repita para o node **"Reply Automatically"** — usando a mesma credencial.

**OpenAI:**
1. Dê um duplo clique no node **"Classify Email with AI"**.
2. Em Credential → "Create New" → cole sua OpenAI API Key (gerada em platform.openai.com/api-keys).
3. O modelo padrão configurado é `gpt-4o-mini` (boa relação custo-benefício). Você pode trocar por outro modelo se precisar de mais precisão.

**Slack:**
1. Dê um duplo clique em qualquer node **"Notify..."** (são quatro: Support, Sales, Finance, General Team).
2. Em Credential → "Create New" → conecte via OAuth com seu workspace do Slack.
3. **Importante:** em cada um dos 4 nodes do Slack, substitua o texto de placeholder `REPLACE_SUPPORT_CHANNEL`, `REPLACE_SALES_CHANNEL`, `REPLACE_FINANCE_CHANNEL` e `REPLACE_GENERAL_CHANNEL` pelo canal real (selecione na lista suspensa do campo).

---

## Passo 5 — Ajustar o limite de confiança (opcional)

Por padrão, o fluxo só responde automaticamente quando a IA tem **80% de confiança ou mais**. Para alterar isso:

1. Abra o node **"Validate AI Response"**.
2. Encontre a linha `highConfidence: confidence >= 0.8` no código.
3. Substitua `0.8` pelo valor desejado (ex.: `0.9` para ser mais conservador).

---

## Passo 6 — Testar o fluxo de trabalho

1. Envie um e-mail de teste simples para a caixa de entrada conectada (ex.: "Qual o horário de atendimento?").
2. Execute o fluxo manualmente no n8n (botão "Execute Workflow", ou o ícone de play no primeiro node).
3. Verifique se:
   - O node "Classify Email with AI" retornou uma categoria coerente.
   - Se a confiança foi alta, a resposta chegou na caixa de e-mail de teste.
   - Se a confiança foi baixa, a notificação chegou no canal certo do Slack.
4. Envie um segundo e-mail de teste, claramente relacionado a "Vendas" ou "Financeiro", para confirmar que o roteamento funciona.
5. Está tudo certo? Clique em **"Active"** para deixar o fluxo rodando automaticamente.

---

## Precauções importantes antes de colocar no ar

⚠️ **Recomendação:** nas primeiras semanas, mantenha o limite de confiança alto (0.85+) e monitore de perto as respostas automáticas. Um e-mail respondido incorretamente pode gerar problemas com um cliente — é melhor escalar demais para um humano no início e ajustar o modelo aos poucos.

## Perguntas frequentes

**A IA pode responder algo errado?**
Sim, por isso existe o campo de confiança. Ajuste o limite (Passo 5) de acordo com o nível de segurança que sua operação exige.

**Posso adicionar mais categorias (ex.: RH, Jurídico)?**
Sim. Edite a lista `VALID_CATEGORIES` no node "Validate AI Response", o prompt no node "Classify Email with AI", e adicione uma nova saída no node "Route by Category".

**O fluxo responde e-mails em qualquer idioma?**
Sim, o modelo de IA detecta e responde no mesmo idioma do e-mail recebido, desde que o prompt não esteja manualmente restrito a um idioma específico.
