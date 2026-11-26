# Prompt da fase 1 para o Lovable

Copie tudo abaixo da linha e cole no modo de criação do Lovable.

---

Crie o **Retoma**, uma aplicação web em português do Brasil que reúne **vitrine pública de produtos, chat próprio em tempo real e painel de atendimento** para uma confeitaria que trabalha com encomendas. Implemente o fluxo completo, com telas funcionais, banco de dados, autenticação e permissões. Quero conseguir testar como comprador em uma janela anônima e como vendedor em outra.

## 1. Objetivo

A confeitaria divulga o link da vitrine pelo Instagram ou WhatsApp. O comprador abre o link no celular, escolhe um doce, pede um orçamento com data e opções e conversa com a loja **dentro do Retoma**. O vendedor responde pelo painel. Nenhum pedido pode se perder: se o comprador fechar a página, ele recebe um e-mail quando a loja responder e reabre a conversa em qualquer aparelho.

| Quem usa | O que precisa conseguir fazer |
|---|---|
| **Comprador** | Ver os produtos sem cadastro, pedir um orçamento, enviar fotos de referência e continuar a conversa depois, mesmo em outro aparelho. |
| **Vendedor** | Saber na hora que chegou mensagem, ver o pedido completo (produto, opções, data, entrega) e responder rápido, inclusive pelo celular. |

**Fora desta versão:** funil Kanban, propostas formais, pagamento online, cálculo de frete, notificação push nativa e qualquer integração com WhatsApp ou Instagram. Não use `wa.me`, Instagram Direct nem widgets de atendimento de terceiros.

## 2. Tecnologia

- **React, TypeScript e Tailwind CSS.**
- **Lovable Cloud** (Supabase/PostgreSQL, Auth, Storage, Realtime e Edge Functions).
- O banco é a fonte oficial dos dados. Armazenamento local só para preferências de interface e para lembrar a origem da visita.
- **Uma única loja nesta versão**, mas **todas as tabelas de dados da loja têm `loja_id`**, e as políticas de acesso filtram por loja. Isso vai permitir várias lojas no futuro sem refazer o banco.
- Envio de e-mails pelo **Resend**, chamado a partir de Edge Functions. A chave fica em segredo do backend, nunca no frontend.
- Tarefas agendadas (alarme de mensagem sem resposta) com **pg_cron** ou um agendamento equivalente do Lovable Cloud.

## 3. Vitrine pública

**Rota:** `/loja/:slug`. Demonstração em `/loja/doceria-horizonte`.

### Catálogo

- Identidade da loja (nome, logo, frase curta), busca visível e filtro por categoria. No celular, os produtos aparecem logo no início da tela.
- Cartão do produto: **foto, nome, preço e ação principal**. Formatos de preço: "R$ 12,00 a unidade", "A partir de R$ 90,00 o kg", "R$ 85,00 o cento" ou **"Sob encomenda"**. Um produto sob encomenda nunca aparece por R$ 0,00.
- Busca por nome ou descrição, combinável com a categoria. Sem resultados, oferecer **Limpar filtros**.
- Página do produto: foto grande, descrição, preço, unidade, **quantidade mínima** (quando houver), **antecedência necessária** ("Encomende com 2 dias de antecedência") e os botões **Pedir orçamento** e **Tirar dúvidas no chat**.

### Pedir orçamento

Formulário em uma etapa, curto e confortável no celular:

1. **Quantidade**, respeitando a quantidade mínima.
2. **Opções do produto**: cada grupo (ex.: "Massa", "Recheio", "Cobertura") é de escolha única ou múltipla, com mínimo e máximo de escolhas quando configurado. **As opções não alteram o preço exibido.**
3. **Data desejada** (obrigatória). O calendário bloqueia datas anteriores à antecedência mínima do produto (ou da loja, se o produto não tiver uma própria) e os dias em que a loja não trabalha.
4. **Entrega ou retirada**, mostrando só as modalidades que a loja oferece. Na entrega, pedir **bairro e endereço** e avisar: "O valor da entrega é combinado no chat".
5. **Observação** curta e, opcionalmente, **foto de referência**.
6. Antes de enviar, mostrar o resumo: produto, quantidade, opções, data e modalidade.

Ao enviar, o pedido aparece no chat como uma **mensagem estruturada** (cartão com foto do produto e os dados acima), e a conversa continua normalmente.

### Chat próprio

- Botão flutuante **Conversar com a loja**. No desktop abre um painel no canto inferior direito; no celular, uma tela inteira com o campo de mensagem visível mesmo com o teclado aberto.
- **Tirar dúvidas no chat** abre o chat com um cartão de contexto (foto, nome e preço do produto). Também é possível iniciar um atendimento geral, sem produto.
- **Primeiro contato:** pedir **nome** e **e-mail** (obrigatórios) e **telefone** (opcional), com a frase: "Usamos seu contato só para responder seu pedido." e um link para a **Política de privacidade**. Sem senha.
- A conversa e o lead só são criados quando o comprador **envia a primeira mensagem ou o pedido de orçamento**. Abrir o widget ou ver produtos não cria nada.
- Mensagens em ordem cronológica, diferenciando comprador e loja, com horário e estados **enviando**, **enviada** e **falhou (tentar de novo)**. O texto digitado é preservado se o envio falhar.
- **Fotos nas duas direções:** JPG, PNG ou WebP, até 5 MB, com pré-visualização e progresso de envio. Ao tocar, a foto abre ampliada.
- **Fora do horário de atendimento**, o chat mostra uma faixa discreta, por exemplo "Respondemos a partir de amanhã, 8h", e continua aceitando mensagens.
- Se o comprador perguntar sobre outro produto, o novo produto entra como contexto da mensagem na mesma conversa.
- Fechar o widget só o recolhe. Ao reabrir ou atualizar a página no mesmo navegador, o histórico continua lá.

### Voltar para a conversa (requisito central)

- Quando a loja responde e o comprador **não está com o chat aberto**, ele recebe um e-mail "A Doceria Horizonte respondeu seu pedido", com um trecho da resposta e o botão **Abrir conversa**.
- O botão é um **link mágico**: ao clicar, em qualquer aparelho ou navegador, o comprador entra autenticado e vê suas conversas com a loja.
- **No máximo um e-mail a cada 30 minutos por conversa.** Mensagens seguidas da loja dentro desse intervalo não geram novos e-mails.
- Na vitrine, um link discreto **"Já fiz um pedido"** permite informar o e-mail e receber um novo link mágico (com limite de tentativas).
- **Segurança:** digitar um e-mail ou telefone **nunca** libera o histórico de ninguém. Só quem clica no link recebido naquele e-mail acessa as conversas associadas a ele. Use sessão anônima do Supabase Auth para o visitante e gere o link mágico no backend (por exemplo, `auth.admin.generateLink`), enviando-o pelo Resend.

### Origem da visita

- A vitrine aceita `?origem=instagram`, `?origem=whatsapp` ou `?origem=outro`. Guarde a origem da primeira visita no navegador.
- Registre **visitas anônimas por dia e por origem** (sem dados pessoais nem cookies de terceiros). O contato e o lead guardam a origem de onde vieram.

## 4. Painel do vendedor

**Rota:** `/painel`, protegida por login (Supabase Auth com e-mail e senha). Navegação: **Atendimento**, **Produtos** e **Configurações**. Abre em **Atendimento**.

### Atendimento em tempo real

No desktop, três colunas. No celular, sequência de telas com botão de voltar.

1. **Lista de conversas:** nome, trecho da última mensagem, horário, responsável e indicador de não lida, ordenada pela atividade mais recente. Busca por nome e filtros **Todas** e **Não lidas**.
2. **Conversa:** histórico completo com os cartões de pedido, fotos e campo de resposta com envio de foto. A resposta aparece no chat do comprador em tempo real. Cada mensagem da loja mostra quem da equipe respondeu.
3. **Ficha do contato:** nome, e-mail, telefone, origem, data do primeiro contato, responsável, **último pedido** (produto, quantidade, opções, data desejada, entrega ou retirada e endereço) e **observações internas**, visíveis só para a equipe. Botão **Excluir dados do contato** com confirmação: apaga dados pessoais, mensagens e fotos daquele contato.

**Responsável automático:** o primeiro vendedor que responde a uma conversa vira o responsável por ela.

**Avisos para o vendedor:**

- Com o painel aberto: **som** e **contador de não lidas no título da aba** quando chega mensagem nova.
- **Alarme por e-mail:** se uma mensagem do comprador ficar sem resposta por **X minutos** (padrão **5**, configurável), enviar um e-mail para o endereço de avisos da loja com o nome do comprador, o trecho da mensagem e o link da conversa. Um único alarme por mensagem pendente.
- O prazo do alarme **só conta dentro do horário de atendimento**. Mensagens recebidas fora do horário geram um alarme quando a loja abre.

### Produtos

- Cadastrar, editar, publicar e despublicar produtos.
- Campos: **nome, categoria, descrição, foto, tipo de preço** (unitário, a partir de, por cento, por kg, sob encomenda), **valor** (quando aplicável), **unidade de venda**, **quantidade mínima** (opcional), **antecedência mínima em dias** (opcional; sem valor, vale o padrão da loja) e **grupos de opções**.
- Grupos de opções: nome do grupo, lista de opções, escolha única ou múltipla e mínimo/máximo de escolhas. Reordenáveis.
- Fotos no Storage, com validação de formato e progresso de upload.
- Gerenciar categorias (criar, renomear, reordenar).
- **Copiar link da vitrine** com seletor de origem (Instagram, WhatsApp, Outro), gerando a URL com `?origem=`. Botão **Abrir minha vitrine**.

### Configurações da loja

- Nome, frase de apresentação e slug.
- **Identidade visual da vitrine:** logo, **cor principal** (seletor de cor com prévia ao vivo e aviso se o contraste ficar ruim) e **foto de capa**.
- **Horário de atendimento** por dia da semana (aberto/fechado, início e fim).
- **Antecedência mínima padrão** em dias.
- **Modalidades:** entrega, retirada ou ambas.
- **Prazo do alarme de "sem resposta"** em minutos (padrão 5).
- **E-mail de avisos** da loja.
- **Equipe:** o dono convida vendedores por e-mail. O convidado cria a senha e entra já vinculado à loja.

### PWA

O painel pode ser **instalado na tela inicial** do celular (manifest, ícone e abertura em tela cheia).

## 5. Regras de backend

Implemente no banco ou em Edge Functions, para funcionar com várias abas e vendedores ao mesmo tempo:

1. **Primeiro envio:** em uma transação, criar ou reaproveitar o contato da sessão, criar a conversa e o **lead** (com etapa `novo_lead`, origem e produto quando houver) e salvar a primeira mensagem ou o pedido. O vendedor vê a conversa sem recarregar a página.
2. **Mesma conversa:** novas mensagens e novos pedidos do mesmo comprador entram na conversa ativa.
3. **Primeira resposta:** a primeira resposta de um vendedor define o responsável e muda a etapa do lead para `primeiro_contato`. (O funil visual chega na próxima fase; o campo já deve existir.)
4. **Envio sem duplicar:** cada envio leva um identificador gerado no cliente. Clique duplo, reconexão ou nova tentativa não pode duplicar mensagem, conversa ou lead.
5. **Reconexão:** mostrar na interface quando a conexão cai. Ao voltar, buscar as mensagens do intervalo e retomar o tempo real.
6. **Validação:** conferir no backend a loja, o produto, a data mínima, as opções escolhidas, o tamanho das mensagens e o tipo e tamanho das fotos. Limitar envios repetitivos.
7. **Métricas:** registrar os eventos `visita` (com origem), `primeira_mensagem`, `email_resposta_enviado` e `retorno_pelo_email` (quando o comprador abre o link mágico), para medir o piloto.

## 6. Dados e acesso

Entidades: **lojas, membros da equipe, categorias, produtos, grupos de opções, opções, contatos, leads, conversas, mensagens, anexos, pedidos de orçamento, eventos de métrica**. Todas com `loja_id`.

- Um pedido de orçamento guarda uma **cópia** do nome do produto, do preço exibido e das opções escolhidas no momento do envio, para o histórico não mudar se o produto for editado.
- **Row Level Security** em todas as tabelas, por loja, por membro da equipe e por dono da sessão do comprador. O mesmo isolamento vale para o Realtime e para o Storage.
- **Leitura pública:** só dados públicos da loja, categorias e produtos publicados.
- O comprador acessa **apenas as próprias conversas, mensagens e fotos**. Nunca vê observações internas, outros contatos ou dados da equipe.
- O comprador não pode se passar por vendedor, mudar etapa ou responsável, nem enviar mensagem em nome da loja.
- Fotos do chat em bucket **privado**, servidas por URL assinada e temporária.
- Chaves secretas (Resend, service role) só no backend.

## 7. Design

Duas experiências com identidades diferentes: **a vitrine tem a cara da loja e dá vontade de encomendar; o painel tem a cara do Retoma e ajuda a responder rápido.**

### Vitrine: identidade da loja

- A vitrine usa a **logo, a cor principal e a foto de capa** configuradas pela loja. A cor principal aparece em botões, links, abas ativas e no botão do chat.
- O restante é fixo, para a vitrine nunca ficar feia com qualquer cor escolhida: superfícies **creme (#FBF6F0)**, texto **cacau (#3B2A25)**, espaçamentos e cantos arredondados. Gere automaticamente as variações da cor principal (hover, fundo suave) e escolha texto claro ou escuro sobre ela pelo contraste.
- **Sempre clara**, sem modo escuro (fotos de comida ficam piores no escuro).
- **Tipografia:** **Fraunces** nos títulos (nome da loja, títulos de seção e nome do produto na página do produto) e **Inter** em todo o resto, inclusive preços, datas e formulários. Sem fontes manuscritas ou cursivas.
- **Catálogo:** grade de **2 colunas no celular** e **3 a 4 no desktop**, com foto quadrada, nome e preço. Categorias em **abas roláveis** no topo, fixas ao rolar. Foto de capa da loja com logo e frase curta acima da grade, sem empurrar os produtos para muito longe.
- Fotos grandes, preços legíveis e o botão de pedido em destaque.
- Rodapé discreto: **"Atendimento por Retoma"**.

### Painel: identidade do Retoma

- Visual **neutro e calmo**: fundo **off-white (#F7F7F5)**, superfícies brancas, bordas e textos secundários em **cinza quente**, texto principal **grafite (#1F2421)** e destaque em **verde-oliva (#5B6B3A)** para ações principais, item ativo do menu e indicadores de não lida.
- **Modo escuro** no painel, seguindo o tema do sistema, com alternância manual em Configurações.
- **Tipografia:** só **Inter**.
- Navegação lateral no desktop e inferior no celular, composição compacta, com destaque para não lidas e para a conversa aberta.

### Regras gerais

- Cores de apoio só para estados (sucesso, erro, atenção), iguais nas duas experiências.
- Português do Brasil em toda a interface: moeda **R$ 180,00**, quantidades **1.000 unidades**, datas **sáb., 14/12**.
- Rótulos curtos, sem slogans genéricos nem métricas decorativas. Explicações só onde há decisão real (para que serve o contato, a diferença entre "a partir de" e preço fechado).
- Contraste adequado, foco visível, rótulos em todos os campos, navegação por teclado e alvos de toque confortáveis.
- Estados vazios, carregamento, erro e sucesso. Nunca mostrar sucesso antes de a operação ser salva.
- Página **/privacidade** com uma política de privacidade simples: dados coletados, finalidade (responder pedidos), armazenamento e como pedir a exclusão.

## 8. Dados de demonstração

Loja **Doceria Horizonte** (slug `doceria-horizonte`), com cor principal **terracota suave (#C8664B)**, logo simples e foto de capa de uma mesa de doces, horário de terça a sábado, das 9h às 18h, antecedência padrão de 2 dias e entrega e retirada habilitadas.

Categorias e produtos, com fotos coerentes:

- **Bolos:** Bolo de aniversário (a partir de R$ 90,00 o kg, mínimo 1 kg, antecedência de 3 dias, grupos Massa, Recheio e Cobertura), Bolo de pote (R$ 12,00 a unidade).
- **Docinhos:** Brigadeiro gourmet (R$ 150,00 o cento, mínimo 25 unidades, grupo Sabores de múltipla escolha), Bem-casado (R$ 4,50 a unidade, mínimo 50).
- **Kits festa:** Kit festa para 20 pessoas (a partir de R$ 220,00), Mesa de doces personalizada (sob encomenda).

Crie algumas conversas fictícias, marcadas discretamente como exemplo, e um vendedor de demonstração.

**Cenário principal**, reproduzível com um visitante novo:

1. Ana abre `/loja/doceria-horizonte?origem=instagram` e entra em **Bolo de aniversário**.
2. Toca em **Pedir orçamento**: 2 kg, massa branca, recheio de ninho com morango, cobertura de chantilly, **retirada no próximo sábado**, observação "É para o aniversário da minha filha" e uma **foto de referência**.
3. Informa nome e e-mail e envia. O vendedor vê a conversa chegar com o cartão do pedido, o som toca e o contador da aba sobe.
4. Ana fecha a página. O vendedor responde "Que lindo! Conseguimos sim. Fica R$ 180,00, pode ser?". Ana recebe o e-mail, toca em **Abrir conversa** em outro navegador e vê a resposta.
5. Se ninguém responder em 5 minutos (dentro do horário), o e-mail de alarme chega para a loja.

## 9. Critérios de aceite

Antes de concluir, percorra a aplicação de verdade e corrija o que falhar:

- Vitrine em janela anônima ou celular e painel autenticado em outra janela. Mensagens e fotos chegam nas duas pontas em tempo real.
- Abrir e fechar o widget sem enviar nada **não cria** contato, conversa nem lead.
- Um pedido de orçamento cria **um contato, uma conversa, um lead e um pedido**, com loja, produto, opções, data e origem corretos.
- O calendário não aceita datas antes da antecedência nem em dias fechados, e o backend recusa essas datas mesmo se o frontend for burlado.
- Clique duplo e nova tentativa de envio não duplicam nada.
- Com o chat do comprador fechado, a resposta do vendedor gera **um** e-mail; várias respostas em 30 minutos não geram mais e-mails. O link mágico abre a conversa em outro navegador.
- O alarme de "sem resposta" respeita o prazo configurado e o horário de atendimento.
- Dois compradores independentes: cada um vê só a própria conversa. Digitar o e-mail de outra pessoa não dá acesso ao histórico dela.
- Atualizar as duas janelas mantém conversa, pedido, responsável e fotos.
- **Excluir dados do contato** remove os dados pessoais, as mensagens e as fotos.
- O painel instala como app no celular e é utilizável com uma mão.

**Entrega:** informe como abrir a vitrine e o painel, como acessar o vendedor de demonstração, como testar o cenário da Ana e quais etapas dependem de configuração manual no Lovable Cloud (chave do Resend, domínio de envio de e-mail, agendamento do alarme).
