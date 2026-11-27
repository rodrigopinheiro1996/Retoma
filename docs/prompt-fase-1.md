# Prompt da fase 1 para o Lovable

Copie tudo abaixo da linha e cole no modo de criação do Lovable.

---

Crie o **Retoma**, um CRM de vendas para pequenos negócios, em português do Brasil. Cada negócio tem sua **vitrine pública**, sua **equipe** e seus **dados**, isolados dos outros. O Retoma ajuda a loja a **receber pedidos de orçamento, conversar com os compradores e acompanhar cada orçamento até o fechamento**.

O piloto é com uma confeitaria. Implemente o fluxo completo, com telas funcionais, banco de dados, autenticação e permissões. Quero testar como compradora em uma janela anônima e como vendedora em outra.

## 1. O fluxo que precisa funcionar

1. A Ana abre a vitrine da loja no celular, escolhe um bolo e toca em **Pedir orçamento**. Informa **nome, e-mail, quantidade, data da festa e observações** e envia.
2. O pedido aparece no painel da loja. O Retoma salva o orçamento e abre **uma conversa vinculada a esse orçamento**. A equipe vê quais orçamentos ainda **aguardam resposta**.
3. Uma vendedora responde pelo painel. Na **primeira resposta**, ela **assume o atendimento** e o orçamento passa de **Novo pedido** para **Em atendimento**. Depois, a equipe marca o orçamento como **Orçamento enviado**, **Confirmado** ou **Perdido**.
4. A Ana fecha a página. Quando a loja responde, ela recebe um e-mail com o botão **Abrir conversa**. Ela abre o link **em outro aparelho** e continua a mesma conversa, **sem criar senha**.

**Fora desta versão:** quadro Kanban, fila de retornos, proposta formal com pagamento, cálculo de frete, notificação push nativa, cadastro aberto de novas lojas e qualquer integração com WhatsApp ou Instagram. Não use `wa.me`, Instagram Direct nem widgets de atendimento de terceiros.

## 2. Tecnologia

- **React, TypeScript e Tailwind CSS.**
- **Lovable Cloud** (Supabase/PostgreSQL, Auth, Storage, Realtime e Edge Functions).
- O banco é a fonte oficial dos dados. Armazenamento local só para preferências de interface e para lembrar a origem da visita.
- **Várias lojas desde o início:** todas as tabelas de dados de negócio têm `loja_id`, e as políticas de acesso isolam cada loja.
- E-mails pelo **Resend**, chamado em Edge Functions. Chaves só em segredos do backend.
- Tarefas agendadas (alarme de orçamento sem resposta) com **pg_cron** ou equivalente do Lovable Cloud.

## 3. Lojas, equipe e acesso

- Uma **loja** tem nome, slug, logo, cor principal, foto de capa, horário de atendimento e configurações.
- **Membros** ligam usuários a lojas, com papel **dono** ou **vendedora**. Uma pessoa pode pertencer a mais de uma loja; nesse caso o painel mostra um seletor de loja.
- O **dono** convida vendedoras por e-mail em **Configurações › Equipe**. A convidada cria a senha e entra vinculada à loja.
- **Criação de lojas:** nesta versão, sem cadastro público. Uma rota **`/admin`**, acessível só a usuários marcados como **administradores da plataforma**, permite criar uma loja (nome e slug) e convidar o dono por e-mail.
- **Row Level Security** em todas as tabelas, por loja e por vínculo de membro. O mesmo isolamento vale para Realtime e Storage. Teste com duas lojas.

## 4. Vitrine pública

**Rota:** `/loja/:slug`. Demonstração em `/loja/doceria-horizonte`.

### Catálogo

- Capa da loja com logo e frase curta, busca visível e categorias em abas roláveis. No celular, os produtos aparecem logo no início.
- Grade de **2 colunas no celular** e **3 a 4 no desktop**. Cartão: **foto quadrada, nome e preço**. Formatos: "R$ 12,00 a unidade", "A partir de R$ 90,00 o kg", "R$ 150,00 o cento" ou **"Sob encomenda"**. Nunca R$ 0,00.
- Busca por nome ou descrição, combinável com a categoria. Sem resultados, **Limpar filtros**.
- Página do produto: foto grande, descrição, preço, unidade, **quantidade mínima** (quando houver), **antecedência necessária** ("Encomende com 3 dias de antecedência") e o botão **Pedir orçamento**.

### Pedir orçamento

Formulário curto, em uma tela, confortável no celular:

1. **Quantidade**, respeitando a quantidade mínima.
2. **Opções do produto** (ex.: Massa, Recheio, Cobertura), de escolha única ou múltipla, quando o produto tiver. **Não alteram o preço.**
3. **Data da festa** (obrigatória). O calendário bloqueia datas antes da antecedência mínima (do produto ou, se não houver, da loja) e dias em que a loja não trabalha.
4. **Entrega ou retirada**, só com as modalidades que a loja oferece. Na entrega, pedir **bairro e endereço** e avisar: "O valor da entrega é combinado no chat".
5. **Observações** e, opcionalmente, uma **foto de referência**.
6. **Nome e e-mail** (obrigatórios) e **telefone** (opcional), com a frase "Usamos seu contato só para responder seu pedido." e link para a **Política de privacidade**. Sem senha.
7. Resumo antes de enviar: produto, quantidade, opções, data e modalidade.

Ao enviar, abrir a tela da conversa desse orçamento, com o **cartão do pedido** no topo (foto do produto e os dados acima) e a mensagem "Recebemos seu pedido. A loja vai responder por aqui e avisar no seu e-mail."

### Conversa do orçamento

- **Cada orçamento tem a sua conversa.** Não existe chat solto sem pedido nesta versão.
- No celular, tela inteira com o campo de mensagem visível mesmo com o teclado aberto. No desktop, painel lateral.
- Mensagens em ordem cronológica, diferenciando cliente e loja, com horário e estados **enviando**, **enviada** e **falhou (tentar de novo)**. O texto digitado é preservado se o envio falhar.
- **Fotos nas duas direções:** JPG, PNG ou WebP, até 5 MB, com pré-visualização e progresso. Ao tocar, a foto amplia.
- Mostrar a **etapa atual** do orçamento em linguagem da cliente: "Aguardando a loja", "Em atendimento", "Orçamento enviado", "Pedido confirmado", "Encerrado".
- **Fora do horário de atendimento**, uma faixa discreta: "Respondemos a partir de amanhã, 9h". As mensagens continuam sendo aceitas.
- Um botão **Meus pedidos** lista os orçamentos da cliente naquela loja, para ela voltar a qualquer um.

### Voltar para a conversa (requisito central)

- Quando a loja responde e a cliente **não está com a conversa aberta**, ela recebe um e-mail "A Doceria Horizonte respondeu seu pedido", com um trecho da resposta e o botão **Abrir conversa**.
- O botão é um **link mágico**: em qualquer aparelho ou navegador, a cliente entra autenticada e cai direto na conversa daquele orçamento.
- **No máximo um e-mail a cada 30 minutos por orçamento.**
- Na vitrine, o link **"Já fiz um pedido"** pede o e-mail e envia um novo link mágico (com limite de tentativas).
- **Segurança:** digitar um e-mail ou telefone **nunca** libera o histórico. Só quem clica no link enviado àquele e-mail acessa os orçamentos dele. Use sessão anônima do Supabase Auth para a visitante e gere o link mágico no backend (por exemplo, `auth.admin.generateLink`), enviando-o pelo Resend.

### Origem da visita

- A vitrine aceita `?origem=instagram`, `?origem=whatsapp` ou `?origem=outro`. Guarde a origem da primeira visita no navegador.
- Registre **visitas anônimas por dia e por origem**, sem dados pessoais nem cookies de terceiros. O orçamento guarda a origem.

## 5. Painel da loja

**Rota:** `/painel`, com login por e-mail e senha. Navegação: **Orçamentos**, **Produtos** e **Configurações**. Abre em **Orçamentos**.

### Orçamentos (tela principal: caixa de conversas)

No desktop, três colunas. No celular, sequência de telas com botão de voltar.

1. **Lista de orçamentos**, cada item com: nome da cliente, produto, data da festa, etapa, responsável, trecho da última mensagem, horário e indicador de não lida. Ordenada pela atividade mais recente.
   - Filtros: **Aguardando resposta** (última mensagem é da cliente), **Novo pedido**, **Em atendimento**, **Orçamento enviado**, **Confirmado**, **Perdido**, **Todos**. Abre em **Aguardando resposta**, com o número de itens em cada filtro.
   - Busca por nome da cliente ou produto.
2. **Conversa** com o cartão do pedido no topo, histórico, fotos e campo de resposta com envio de foto. A resposta chega à cliente em tempo real. Cada mensagem da loja mostra quem respondeu.
3. **Ficha do orçamento**: cliente (nome, e-mail, telefone, origem), produto, quantidade, opções, data da festa, entrega ou retirada com endereço, observações, foto de referência, **etapa**, **responsável**, **valor combinado** (opcional, preenchido pela equipe) e **observações internas** (só a equipe vê). Também mostra os **outros orçamentos** da mesma cliente.

**Etapas e responsável:**

| Etapa | Quando |
|---|---|
| **Novo pedido** | A cliente enviou o pedido. |
| **Em atendimento** | Automática na primeira resposta da equipe. Quem respondeu vira a responsável. |
| **Orçamento enviado** | A equipe marca ao passar o valor. |
| **Confirmado** | A equipe marca quando o pedido fecha. |
| **Perdido** | A equipe marca, com motivo: Sem agenda na data, Preço, Sem resposta da cliente, Desistiu, Outro. |

- Ações na ficha: **Mudar etapa** (seletor, funciona no celular e no teclado) e **Trocar responsável**.
- Registrar cada mudança de etapa e de responsável com data, hora, valor anterior, novo valor e autor. Mostrar esse histórico na ficha.
- Se uma atualização falhar, voltar ao valor anterior na tela e explicar.

**Avisos para a equipe:**

- Com o painel aberto: **som** e **contador de não lidas no título da aba**.
- **Alarme por e-mail:** se a mensagem de uma cliente ficar sem resposta por **X minutos** (padrão **5**, configurável), enviar um e-mail para o endereço de avisos da loja com nome, produto, trecho e link do orçamento. Um alarme por mensagem pendente.
- O prazo só conta **dentro do horário de atendimento**. Mensagens fora do horário geram o alarme na abertura.
- Botão **Excluir dados da cliente**, com confirmação: apaga dados pessoais, mensagens e fotos dela naquela loja.

### Produtos

- Cadastrar, editar, publicar e despublicar.
- Campos: **nome, categoria, descrição, foto, tipo de preço** (unitário, a partir de, por cento, por kg, sob encomenda), **valor**, **unidade**, **quantidade mínima** (opcional), **antecedência em dias** (opcional; sem valor, vale o padrão da loja) e **grupos de opções** (nome, opções, escolha única ou múltipla, mínimo e máximo).
- Fotos no Storage, com validação e progresso.
- Categorias: criar, renomear, reordenar.
- **Copiar link da vitrine** com seletor de origem (Instagram, WhatsApp, Outro) e **Abrir minha vitrine**.

### Configurações

- Nome, frase de apresentação e slug.
- **Identidade visual da vitrine:** logo, **cor principal** (com prévia ao vivo e aviso de contraste ruim) e **foto de capa**.
- **Horário de atendimento** por dia da semana.
- **Antecedência padrão** em dias.
- **Modalidades:** entrega, retirada ou ambas.
- **Prazo do alarme** em minutos (padrão 5) e **e-mail de avisos**.
- **Equipe:** convidar, listar e remover vendedoras (só o dono).

O painel pode ser **instalado na tela inicial** do celular (PWA com manifest e ícone).

## 6. Regras de backend

Em banco ou Edge Functions, para funcionar com várias abas e vendedoras ao mesmo tempo:

1. **Envio do pedido:** em uma transação, criar ou reaproveitar o **contato** (por sessão da visitante e loja), criar o **orçamento** em **Novo pedido** com origem e uma **cópia** do nome do produto, preço exibido e opções escolhidas, criar a **conversa** vinculada e salvar o cartão do pedido como primeira mensagem. A equipe vê o orçamento sem recarregar.
2. **Primeira resposta:** define a responsável e muda **Novo pedido → Em atendimento**, uma única vez, mesmo com duas vendedoras respondendo ao mesmo tempo.
3. **Mudanças de etapa** só pela equipe da loja, sempre registradas no histórico.
4. **Envio sem duplicar:** cada envio leva um identificador gerado no cliente. Clique duplo ou nova tentativa não duplicam pedido, conversa ou mensagem.
5. **Reconexão:** mostrar quando a conexão cai; ao voltar, buscar as mensagens do intervalo e retomar o tempo real.
6. **Validação:** loja, produto publicado, quantidade mínima, data mínima, dias de funcionamento, opções, tamanho das mensagens, tipo e tamanho das fotos. Limitar envios repetitivos.
7. **Métricas:** registrar `visita` (com origem), `pedido_enviado`, `email_resposta_enviado` e `retorno_pelo_email` (quando a cliente abre o link mágico).

## 7. Dados e privacidade

Entidades: **lojas, membros, administradores da plataforma, categorias, produtos, grupos de opções, opções, contatos, orçamentos, conversas, mensagens, anexos, histórico de etapas, eventos de métrica**. Todas as de negócio com `loja_id`.

- **Leitura pública:** só dados públicos da loja, categorias e produtos publicados.
- A cliente acessa **apenas os próprios orçamentos, mensagens e fotos**. Nunca vê observações internas, valor combinado interno, histórico de etapas, outros contatos ou dados da equipe.
- A cliente não pode mudar etapa ou responsável nem enviar mensagem em nome da loja.
- Fotos em bucket **privado**, servidas por URL assinada temporária.
- Página **/privacidade** simples: dados coletados, finalidade, armazenamento e como pedir a exclusão.

## 8. Design

**A vitrine tem a cara da loja e dá vontade de encomendar. O painel tem a cara do Retoma e ajuda a responder rápido.**

### Vitrine

- Usa **logo, cor principal e foto de capa** da loja. A cor principal aparece em botões, links, abas ativas e destaques.
- Fixos: superfícies **creme (#FBF6F0)**, texto **cacau (#3B2A25)**, espaçamentos e cantos arredondados. Gerar variações da cor principal e escolher texto claro ou escuro sobre ela pelo contraste.
- **Sempre clara.**
- **Fraunces** nos títulos (nome da loja, seções, nome do produto) e **Inter** no resto, inclusive preços, datas e formulários.
- Rodapé discreto: **"Atendimento por Retoma"**.

### Painel

- Fundo **off-white (#F7F7F5)**, superfícies brancas, **cinza quente** nos secundários, texto **grafite (#1F2421)** e destaque **verde-oliva (#5B6B3A)** para ações principais, item ativo e não lidas.
- **Modo escuro** seguindo o sistema, com alternância manual.
- Só **Inter**.
- Menu lateral no desktop e inferior no celular. Composição compacta.

### Regras gerais

- Cores de apoio só para estados (sucesso, erro, atenção). Etapas com rótulo em texto, nunca só por cor.
- Português do Brasil: **R$ 180,00**, **1.000 unidades**, **sáb., 14/12**.
- Rótulos curtos, sem slogans nem métricas decorativas.
- Contraste adequado, foco visível, rótulos nos campos, navegação por teclado, alvos de toque confortáveis.
- Estados vazios, carregamento, erro e sucesso. Nunca mostrar sucesso antes de salvar.

## 9. Dados de demonstração

- **Administrador da plataforma** de demonstração.
- Loja **Doceria Horizonte** (`doceria-horizonte`): cor **terracota suave (#C8664B)**, logo simples, capa com mesa de doces, terça a sábado das 9h às 18h, antecedência padrão de 2 dias, entrega e retirada. **Uma dona e uma vendedora** de demonstração.
- Produtos com fotos coerentes:
  - **Bolos:** Bolo de aniversário (a partir de R$ 90,00 o kg, mínimo 1 kg, antecedência de 3 dias, grupos Massa, Recheio e Cobertura), Bolo de pote (R$ 12,00 a unidade).
  - **Docinhos:** Brigadeiro gourmet (R$ 150,00 o cento, mínimo 25, grupo Sabores de múltipla escolha), Bem-casado (R$ 4,50 a unidade, mínimo 50).
  - **Kits festa:** Kit festa para 20 pessoas (a partir de R$ 220,00), Mesa de doces personalizada (sob encomenda).
- Alguns orçamentos fictícios em etapas diferentes, marcados discretamente como exemplo.
- Uma **segunda loja** mínima (ex.: **Ateliê Aurora**, com 2 produtos) para testar o isolamento.

## 10. Critérios de aceite

Percorra a aplicação de verdade e corrija o que falhar:

- **Cenário da Ana:**
  1. Ana abre `/loja/doceria-horizonte?origem=instagram`, escolhe **Bolo de aniversário** e pede 2 kg, massa branca, recheio de ninho com morango, cobertura de chantilly, **retirada no próximo sábado**, observação "É para o aniversário da minha filha", foto de referência, nome e e-mail.
  2. O orçamento aparece no painel em **Novo pedido** e em **Aguardando resposta**, com som e contador na aba.
  3. Ana fecha a página. A vendedora responde "Que lindo! Conseguimos sim. Fica R$ 180,00, pode ser?". O orçamento vai para **Em atendimento** e ela vira a responsável.
  4. Ana recebe **um** e-mail, abre **Abrir conversa** em outro navegador, vê a resposta e escreve "Pode sim!".
  5. A vendedora marca **Orçamento enviado** e depois **Confirmado**. O histórico mostra as duas mudanças.
- Um pedido cria **um contato, um orçamento, uma conversa** e o cartão do pedido, com loja, produto, opções, data e origem corretos.
- Calendário e backend recusam data antes da antecedência e em dia fechado.
- Clique duplo e nova tentativa não duplicam nada.
- Várias respostas em 30 minutos geram só um e-mail.
- Duas vendedoras respondendo ao mesmo tempo: só uma vira responsável, e a etapa muda uma vez.
- Alarme de "sem resposta" respeita o prazo e o horário.
- Duas visitantes: cada uma vê só os próprios orçamentos. Digitar o e-mail de outra pessoa não dá acesso.
- Duas lojas: a equipe de uma não vê nada da outra, nem pelo Realtime nem pelo Storage.
- **Excluir dados da cliente** remove dados pessoais, mensagens e fotos.
- O painel instala como app no celular.

**Entrega:** diga como abrir a vitrine, o painel e o `/admin`, quais são os logins de demonstração, como rodar o cenário da Ana e o que depende de configuração manual no Lovable Cloud (chave do Resend, domínio de envio, agendamento do alarme).
