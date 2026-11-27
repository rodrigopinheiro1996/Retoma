# Retoma: decisões do produto

> **Mudança de direção (27/11/2026):** o piloto passou a ser um CRM de **cartões, tela Hoje e agenda** para vários segmentos (salões, assistências, oficinas, gráficas, lojas, serviços), com WhatsApp por link e sem custo de e-mail ou API. Vitrine, chat próprio e link mágico ficam fora do piloto (aguardando decisão). O prompt atual está em [prompt-fase-1.md](prompt-fase-1.md) e as propostas em [piloto-propostas.html](piloto-propostas.html). O conteúdo abaixo registra o plano anterior.

Resultado da sessão de perguntas (grill-me) sobre o projeto. Cada item registra o que foi decidido e por quê.

## Visão

O Retoma é um **CRM de vendas para pequenos negócios**, vendido como produto. Cada negócio tem sua vitrine, equipe e dados. O Retoma ajuda a loja a **receber pedidos de orçamento, conversar com os compradores e acompanhar cada orçamento até o fechamento**. O primeiro piloto é com uma **confeitaria**.

**Proposta de valor:** nenhum pedido se perde, e a loja sabe em que pé está cada venda. Não pagar a Meta é um bônus, não o argumento principal, porque quem atende pelo app WhatsApp Business comum (sem API) já não paga por mensagem.

## Decisões de base

| # | Decisão | Motivo |
|---|---|---|
| Q1 | **Várias lojas desde a v1** (revisado). Lojas criadas por um `/admin` da plataforma, sem cadastro público no piloto | O Retoma é um produto para vários negócios |
| Q2 | Valor central = **organização** | A economia com a Meta é fraca para quem usa o app comum |
| Q3 | **Ainda não há loja piloto** | A fase 1 serve de demonstração para prospectar |
| Q4 | **E-mail obrigatório** no primeiro contato, com link mágico para reabrir a conversa | O comprador fecha o site e precisa saber que a resposta chegou, em qualquer aparelho |
| Q5 | **React + TypeScript + Tailwind + Lovable Cloud (Supabase)** | O Lovable gera React; chat em tempo real exige backend |
| Q6 | Construir em **3 fases** | Prompts muito grandes no Lovable saem com partes quebradas |
| Q7 | Segmento: **doces e encomendas** | Todo pedido é personalizado e negociado, ou seja, o chat é central |
| Q8 | Construir só a fase 1 e **prospectar com ela**; fase 2 só com uma loja real | Evita construir às cegas |
| Q13 | Critérios de sucesso do piloto (30 dias) | Ver abaixo |

### Critérios de sucesso do piloto

1. **≥ 30%** de quem recebe o link envia mensagem ou pedido.
2. **≥ 50%** de quem recebe resposta com o site fechado volta pelo e-mail.
3. A loja **continua usando após 30 dias** sem insistência.

Os dois primeiros são medidos automaticamente pelo Retoma (visitas por origem, primeira mensagem, retorno pelo e-mail).

## Revisão: fluxo centrado no orçamento

- **Cada orçamento tem a sua conversa.** A cliente começa pelo **Pedir orçamento** na vitrine; não há chat solto sem pedido.
- **Etapas:** Novo pedido → Em atendimento (automática na primeira resposta, que define a responsável) → Orçamento enviado → Confirmado / Perdido (com motivo). Mudança manual pela equipe já na fase 1, com histórico.
- **Tela principal do painel:** caixa de conversas com os orçamentos vinculados, abrindo no filtro **Aguardando resposta**.
- Substitui as etapas antigas (Novo Lead, Primeiro Contato, Proposta, Ganho, Perdido) e os itens das fases 2 e 3 abaixo onde houver conflito.

## Fase 1: vitrine, chat, inbox e produtos

| # | Decisão |
|---|---|
| Q9 | A fase 1 inclui **pedido de orçamento simples** e **cadastro básico de produtos** |
| Q10 | Vários logins de vendedor. **Quem responde primeiro vira o responsável.** Sem "Assumir atendimento" e sem filtro "Minhas" |
| Q11 | E-mail ao comprador **só com o chat fechado**, **no máximo 1 a cada 30 min por conversa**, enviado via **Resend** |
| Q12 | Vendedor: **som e contador na aba** + **e-mail se a mensagem ficar sem resposta** (padrão **5 min**, configurável pela loja) |
| Q14 | **Data desejada obrigatória** no pedido. **Antecedência mínima** com padrão da loja, ajustável por produto e editável |
| Q15 | **Grupos de opções** (sabor, recheio, tamanho), de escolha única ou múltipla, **sem alterar o preço**. Preço "a partir de"; o valor final é combinado no chat |
| Q16 | **Entrega** (bairro + endereço) ou **Retirada**. A loja configura quais oferece. Frete combinado no chat |
| Q17 | **Fotos no chat nas duas direções** (JPG/PNG/WebP, até 5 MB), com acesso restrito à conversa |
| Q18 | **Copiar link com origem** (`?origem=instagram`, `whatsapp`, `outro`). Visitas anônimas contadas por origem; o lead guarda a origem |
| Q19 | Demo: **Doceria Horizonte**, com o cenário do bolo de 2 kg para sábado |
| Q21 | **Horário de atendimento** configurável. Fora do horário o widget avisa quando a loja responde; o alarme de "sem resposta" só conta no expediente |
| Q25 | **LGPD:** frase curta no formulário, página de política de privacidade e botão **Excluir dados do contato** |
| Q26 | Painel instalável como **PWA**. Domínio próprio só quando houver loja piloto |

## Visual

| # | Decisão |
|---|---|
| Q28 | **Vitrine com a identidade da loja** (logo, cor principal e foto de capa configuráveis; o resto fixo). **Painel com a identidade do Retoma.** Rodapé "Atendimento por Retoma" |
| Q29 | Painel **neutro e calmo**: off-white, cinza quente, grafite e destaque **verde-oliva (#5B6B3A)** |
| Q30 | Demo Doceria Horizonte: **terracota suave (#C8664B)**, fundo **creme (#FBF6F0)**, texto **cacau (#3B2A25)** |
| Q31 | **Fraunces** nos títulos da vitrine, **Inter** no resto; painel só com Inter. Sem fontes cursivas |
| Q32 | Catálogo em **grade de 2 colunas no celular** (3 a 4 no desktop), categorias em abas roláveis |
| Q33 | **Modo escuro só no painel**; vitrine sempre clara |

## Fase 2: funil e agenda

- Kanban com as mesmas etapas da fase 1 (Novo pedido, Em atendimento, Orçamento enviado, Confirmado, Perdido).
- **Fila de retornos.**
- Selo **"Parado há X dias"** a partir de 3 dias sem atividade (configurável). Nada se move automaticamente (Q24).
- "Assumir atendimento" e filtro "Minhas", se a loja pedir.
- **Agenda** (Q20): encomendas ganhas agrupadas por data de entrega.

## Fase 3: propostas e fechamento

- Proposta com **valor total, % de sinal e chave Pix em texto**, com botão "Copiar chave" (Q22).
- O comprador envia o comprovante como foto; o vendedor marca **Sinal recebido** e depois **Ganho**.
- Botão **Aceitar proposta** para o comprador (Q23).
- Histórico de etapas e arquivamento de produtos.

## Fora do escopo até o piloto

Notificação push nativa, domínio próprio, preço variável por opção, limite de capacidade por dia, pagamento integrado, cálculo de frete, integração com a API do WhatsApp/Instagram.
