# Retoma: decisões do produto

Resultado da sessão de perguntas (grill-me) sobre o projeto. Cada item registra o que foi decidido e por quê.

## Visão

O Retoma é uma **vitrine com chat próprio e CRM** para **confeitarias e encomendas de doces**. A loja divulga o link da vitrine pelo Instagram ou WhatsApp. O comprador escolhe um produto, pede um orçamento e conversa com a loja dentro do Retoma. O vendedor responde pelo painel e acompanha cada encomenda.

**Proposta de valor:** nenhum pedido se perde, e a loja sabe em que pé está cada venda. Não pagar a Meta é um bônus, não o argumento principal, porque quem atende pelo app WhatsApp Business comum (sem API) já não paga por mensagem.

## Decisões de base

| # | Decisão | Motivo |
|---|---|---|
| Q1 | V1 para **uma única loja**, com `loja_id` em todas as tabelas | Mantém a v1 simples sem impedir que vire SaaS depois |
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

## Fase 2: funil e agenda

- Kanban: **Novo Lead → Primeiro Contato → Proposta → Ganho → Perdido**. O cartão passa para **Primeiro Contato** na primeira resposta do vendedor.
- Selo **"Parado há X dias"** a partir de 3 dias sem atividade (configurável). Nada se move automaticamente (Q24).
- "Assumir atendimento" e filtro "Minhas", se a loja pedir.
- **Agenda** (Q20): encomendas ganhas agrupadas por data de entrega.

## Fase 3: propostas e fechamento

- Proposta com **valor total, % de sinal e chave Pix em texto**, com botão "Copiar chave" (Q22).
- O comprador envia o comprovante como foto; o vendedor marca **Sinal recebido** e depois **Ganho**.
- Botão **Aceitar proposta** para o comprador (Q23).
- **Perdido** exige motivo: Sem agenda na data, Preço, Sem resposta do cliente, Desistiu, Outro (Q23).
- Histórico de etapas e arquivamento de produtos.

## Fora do escopo até o piloto

Notificação push nativa, domínio próprio, preço variável por opção, limite de capacidade por dia, pagamento integrado, cálculo de frete, integração com a API do WhatsApp/Instagram.
