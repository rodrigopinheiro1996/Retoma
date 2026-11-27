# Prompt do piloto para o Lovable

Copie tudo abaixo da linha e cole no modo de criação do Lovable.

---

Crie o **Retoma**, um CRM simples para pequenos negócios, em português do Brasil. Ele organiza **clientes, atendimentos e compromissos em cartões**, para que o dono e a equipe saibam **o que precisam fazer hoje** e **o que está atrasado**. Serve a negócios diferentes: salões, assistências técnicas, oficinas, gráficas, lojas e prestadores de serviço.

Cada negócio tem seu próprio espaço, sua equipe e acesso **apenas aos seus dados**. Implemente telas funcionais, banco de dados, autenticação e permissões.

**Custo zero de operação além da hospedagem:** sem API da Meta, sem Resend ou outro serviço de e-mail, sem ferramenta paga de notificações. Os lembretes aparecem dentro do Retoma. O WhatsApp é aberto por link (`https://wa.me/...`).

## 1. Conceitos

| Conceito | O que é |
|---|---|
| **Negócio** | O espaço de uma empresa: equipe, clientes, cartões, etapas e mensagens prontas. |
| **Cliente** | Pessoa atendida: nome, telefone (WhatsApp), e-mail opcional e observações. Pode ter vários cartões. |
| **Cartão** | Um atendimento: título, cliente, responsável, etapa, observações e valor opcional. Ex.: "Troca de tela – iPhone 12", "Escova + hidratação". |
| **Próximo passo** | Uma tarefa com data e hora ligada a um cartão. Ex.: "Retornar sobre o orçamento" na sexta às 10h. |
| **Agendamento** | Um horário marcado com o cliente, com início e duração. Ex.: serviço na sexta às 14h, 1h30. Pode ter um **lembrete de confirmação** separado (ex.: quinta às 10h). |

**Regra de ouro:** todo cartão aberto deve ter um próximo passo ou um agendamento futuro. Cartão aberto sem nada marcado aparece em destaque como **"Sem próximo passo"**. É assim que nenhum cliente fica esquecido.

## 2. Tecnologia

- **React, TypeScript e Tailwind CSS.**
- **Lovable Cloud** (Supabase/PostgreSQL, Auth e Realtime). Sem Edge Functions pagas nem serviços externos.
- Todas as tabelas de negócio têm `negocio_id`. **Row Level Security** isola cada negócio em consultas, gravações e Realtime.
- Cada negócio tem **fuso horário** (padrão `America/Sao_Paulo`). "Hoje", "atrasado" e a agenda usam esse fuso.
- Atualização em tempo real: se uma pessoa da equipe conclui ou remarca algo, a tela dos outros atualiza sem recarregar.
- Painel instalável como app no celular (**PWA** com manifest e ícone).

## 3. Acesso e equipe

- Login por e-mail e senha (Supabase Auth).
- Papéis: **dono** e **equipe**. A equipe vê e edita todos os cartões do negócio. Só o dono gerencia etapas, mensagens prontas, equipe e dados do negócio.
- O dono convida pessoas por e-mail usando o convite nativo do Supabase Auth. Também pode gerar um **link de convite** para copiar e mandar pelo WhatsApp (sem depender de e-mail).
- Uma pessoa pode estar em mais de um negócio; nesse caso há um seletor de negócio.
- **Criação de negócios:** sem cadastro público no piloto. Uma rota **`/admin`**, só para administradores da plataforma, cria o negócio (nome, segmento, fuso) e gera o link de convite do dono.
- Ao criar o negócio, o **segmento** (Salão, Assistência técnica, Oficina, Gráfica, Loja, Serviços) pré-configura etapas sugeridas e mensagens prontas. Tudo pode ser editado depois.

## 4. Telas

Navegação: **Hoje**, **Quadro**, **Agenda**, **Clientes** e **Configurações**. Menu inferior no celular, lateral no desktop. O app abre em **Hoje**.

Botão **+ Novo** sempre visível (no celular, botão flutuante), com as opções **Atendimento** e **Agendamento**.

### 4.1 Hoje (tela principal)

Lista de compromissos de próximos passos, lembretes de confirmação e agendamentos, em seções:

1. **Atrasados**: pendentes com data/hora já passada, do mais antigo para o mais recente, com "há 2 dias" ou "há 3h".
2. **Hoje**: em ordem de horário. Itens sem hora ficam no topo como "Hoje, qualquer horário".
3. **Agendamentos de hoje que já passaram e ainda não foram marcados**: "Como foi?" com **Realizado**, **Faltou** e **Remarcar**.
4. **Próximos 7 dias**: agrupados por dia, recolhidos por padrão no celular.
5. **Sem próximo passo**: cartões abertos sem nada marcado.

Cada item mostra: tipo (ícone e rótulo: Retorno, Confirmação, Agendamento), título da ação, cliente, título do cartão, hora, responsável e etapa.

**Ações direto no item, sem abrir o cartão:**

- **WhatsApp**: abre o WhatsApp do cliente com a mensagem pronta do tipo da ação (ver 4.7).
- **Concluir**: abre uma folha curta com:
  - **O que aconteceu?** Atalhos por tipo (Confirmação: *Confirmado*, *Pediu para remarcar*, *Cancelou*, *Sem resposta*; Retorno: *Aprovou*, *Vai pensar*, *Recusou*, *Sem resposta*) e campo de texto opcional.
  - **Próximo passo**: atalhos *Amanhã*, *Em 2 dias*, *Próxima semana*, *Escolher data e hora* e *Nenhum*. O título vem sugerido conforme o resultado (ex.: *Sem resposta* sugere "Tentar de novo" em 2 horas).
  - **Mudar etapa** (opcional), com a etapa sugerida pelo resultado (ex.: *Aprovou* sugere "Em andamento").
  - Um único botão **Salvar**. Concluir, registrar o resultado, agendar o próximo passo e mudar a etapa acontecem juntos, numa operação só.
  - Se a pessoa escolher *Nenhum* e o cartão continuar aberto, avisar: "Este cartão vai ficar sem próximo passo".
- **Remarcar**: atalhos *Daqui a 1h*, *Amanhã neste horário*, *Próxima segunda* e *Escolher*. Registra no histórico a data antiga e a nova.

**Filtros e contadores no topo:**

- **Responsável**: *Minhas* (padrão), *Todos* ou uma pessoa.
- Contadores clicáveis: **Atrasados**, **Para hoje**, **Concluídos hoje** e **Sem próximo passo**.
- Busca por nome ou telefone do cliente.

**Visão da equipe** (só para o dono, um botão em Hoje): tabela por pessoa com atrasados, para hoje, concluídos hoje e concluídos na semana. Clicar numa pessoa filtra a tela.

**Avisos sem custo:**

- O número de atrasados + para hoje aparece no **título da aba** e no **ícone do app instalado** (Badging API, quando o navegador suporta).
- Com o app aberto, quando um compromisso chega na hora, aparece um aviso na tela e, se a pessoa permitir, uma **notificação do navegador** (Notification API local, sem servidor de push).

### 4.2 Quadro

- Colunas = etapas do negócio. Etapas padrão: **Novo**, **Em andamento**, **Aguardando cliente** e **Concluído**.
- Cada etapa tem um **tipo**: *aberta*, *concluída* ou *perdida*. Os contadores e a regra "sem próximo passo" usam o tipo, não o nome. O dono pode criar uma etapa do tipo *perdida* (ex.: "Cancelado").
- O dono **renomeia, cria, reordena e arquiva** etapas em Configurações. Não é possível apagar uma etapa com cartões: é preciso movê-los antes.
- Cartão no quadro: título, cliente, responsável (avatar com iniciais), valor (se houver) e o **próximo compromisso** com data. Selo vermelho **Atrasado** ou selo cinza **Sem próximo passo**.
- Arrastar entre colunas no desktop. No celular e no teclado, seletor **Mover para**.
- Colunas do tipo *concluída* e *perdida* mostram só os últimos 30 dias, com link "Ver todos".
- Topo: busca, filtro por responsável e total de valor por coluna (apenas valores preenchidos).

### 4.3 Ficha do cartão

Abre como painel lateral no desktop e tela inteira no celular.

- **Cabeçalho**: título editável, cliente (link para a ficha), botões **WhatsApp** e **Ligar** (`tel:`).
- **Campos**: etapa, responsável, valor (opcional, em R$) e observações.
- **Compromissos**: próximos passos e agendamentos pendentes, com Concluir e Remarcar, mais **+ Próximo passo** e **+ Agendamento**.
- **Histórico** (mais recente primeiro): criação, mudanças de etapa, responsável e valor (com antes e depois), compromissos criados, remarcados e concluídos com o resultado, WhatsApp aberto e notas. Cada linha mostra **quem** e **quando**.
- Campo **Adicionar nota** no histórico, para registrar o que foi combinado.

### 4.4 Agendamento e lembrete de confirmação

- Campos: cliente, cartão (existente ou novo, criado no mesmo formulário), título do serviço, **data e hora de início**, **duração** (atalhos 30 min, 1h, 1h30, 2h), responsável e observações.
- **Lembrete de confirmação**, ligado por padrão para os segmentos de horário marcado: sugestão **no dia anterior às 10h** (configurável por negócio). Se o agendamento for para o dia seguinte e já passou das 10h, sugerir "Hoje, daqui a 1h". Dá para mudar ou desligar.
- Estados do agendamento: **Marcado**, **Confirmado**, **Realizado**, **Faltou** e **Cancelado**.
- Concluir a confirmação com *Confirmado* marca o agendamento como Confirmado. *Cancelou* marca como Cancelado e pergunta se quer agendar outro horário.
- **Conflito de horário**: ao marcar, avisar se o responsável já tem outro agendamento que se sobrepõe ("Mariana já tem horário das 14h às 15h"). Apenas avisar, sem bloquear.

### 4.5 Agenda

- Visão **Dia** (padrão no celular) e **Semana** (padrão no desktop), com setas e botão **Hoje**.
- Mostra **agendamentos** em blocos pela duração, coloridos pelo estado (com rótulo em texto). Um botão liga e desliga os **próximos passos** como marcadores menores.
- Filtro por responsável. Na semana, uma linha por responsável quando houver mais de uma pessoa (opcional).
- Tocar num horário vazio cria um agendamento naquele horário. Tocar num bloco abre o cartão.

### 4.6 Clientes

- Lista com busca por **nome ou telefone** (com ou sem máscara, com ou sem DDD 55). Mostra cartões abertos e o próximo compromisso.
- **Ficha do cliente**: dados, botões WhatsApp e Ligar, lista de cartões (abertos e encerrados) e **histórico unificado** de todos os cartões.
- **Botão + Novo atendimento** na ficha, já com o cliente preenchido.
- **Telefone**: aceitar `(11) 98765-4321`, `11987654321` ou `+55 11 98765-4321` e salvar normalizado (E.164). Ao cadastrar, se o telefone já existir no negócio, mostrar "Este telefone já é de Carlos Souza" com **Usar este cliente**.

### 4.7 WhatsApp por link

- O botão abre `https://wa.me/55DDDNUMERO?text=MENSAGEM` em nova aba (no celular, abre o app).
- **Mensagens prontas** por tipo (Confirmação de horário, Retorno de orçamento, Aviso de serviço pronto, Genérica), editáveis pelo dono, com variáveis: `{cliente}`, `{primeiro_nome}`, `{servico}`, `{data}`, `{hora}`, `{valor}`, `{negocio}`, `{responsavel}`.
- Antes de abrir, mostrar a mensagem preenchida num campo editável, com **Abrir WhatsApp** e a opção de trocar o modelo. A pessoa revisa e envia no próprio WhatsApp.
- Registrar no histórico "WhatsApp aberto por Mariana (Confirmação de horário)".
- Ao voltar para o Retoma depois de abrir o WhatsApp a partir de um compromisso, mostrar a folha **Concluir** daquele compromisso (pode ser fechada).
- Sem telefone cadastrado, o botão vira **Adicionar telefone**.

### 4.8 Novo atendimento

Um único formulário curto:

1. **Cliente**: busca por nome ou telefone; se não achar, criar ali mesmo com nome e telefone.
2. **Título** do atendimento.
3. **Valor** (opcional).
4. **Etapa** (padrão: a primeira etapa aberta) e **Responsável** (padrão: quem está criando).
5. **Próximo passo**: atalhos *Hoje mais tarde*, *Amanhã*, *Em 2 dias*, *Escolher*, ou **Marcar agendamento**. Pode ficar vazio, mas o cartão vai para "Sem próximo passo".
6. **Observações**.

Salvar volta para a tela de origem.

### 4.9 Configurações (dono)

- Dados do negócio: nome, segmento e fuso.
- **Etapas**: renomear, criar, reordenar, definir tipo e arquivar.
- **Mensagens prontas**: editar os modelos com prévia preenchida.
- **Lembrete de confirmação padrão**: ligado ou desligado, quantos dias antes e a que horas.
- **Equipe**: convidar (e-mail ou link), mudar papel e remover.

## 5. Regras de backend

1. Concluir com próximo passo e mudança de etapa é **uma transação** (função no banco). Se falhar, nada muda e a tela explica.
2. Toda mudança de etapa, responsável, valor e compromisso grava uma linha de **histórico** com autor, data, valor anterior e novo valor (trigger no banco).
3. Cada envio leva um identificador gerado no cliente. Clique duplo não cria cartão, cliente ou compromisso duplicado.
4. Atualizações concorrentes: se duas pessoas concluem o mesmo compromisso, a segunda vê "Já concluído por Mariana às 10:42".
5. Validar `negocio_id` e permissões no banco. A equipe não altera etapas nem configurações; ninguém acessa dados de outro negócio.
6. Datas gravadas em UTC; exibidas no fuso do negócio.

## 6. Design

**Objetivo: em 5 segundos a pessoa sabe o que fazer agora.**

- Fundo **off-white (#F7F7F5)**, superfícies brancas, **cinza quente** nos secundários, texto **grafite (#1F2421)** e destaque **verde-oliva (#5B6B3A)** em ações principais, item ativo e Concluído.
- **Atrasado** em vermelho tijolo, **Hoje** em âmbar e **Confirmado** em verde, sempre acompanhados de rótulo em texto.
- **Inter** em toda a interface. **Modo escuro** seguindo o sistema, com alternância manual.
- No celular: botões de ação com pelo menos 44 px, ações principais ao alcance do polegar e folhas que sobem de baixo para Concluir e Remarcar.
- Português do Brasil: **R$ 650,00**, **sex., 14/03 às 14:00**, "há 2 dias", "amanhã".
- Rótulos curtos, sem slogans nem métricas decorativas.
- Estados vazios úteis. Ex.: em Hoje, "Nada atrasado. Próximo compromisso: amanhã às 9h, Carlos Souza".
- Contraste adequado, foco visível, rótulos nos campos e navegação por teclado. Tudo que é feito arrastando também pode ser feito por botão.
- Carregamento, erro e sucesso tratados. Nunca mostrar sucesso antes de salvar.

## 7. Dados de demonstração

Crie dois negócios, com datas **relativas ao dia atual** (para a demonstração sempre ter itens em Atrasados, Hoje e Próximos). Em `/admin`, um botão **Recriar dados de demonstração** apaga e recria esses dois negócios com datas atualizadas.

**Assistência técnica "Conserta Já"** (dono: Rafael; equipe: Bruno)
- Etapas: Novo, Em análise, Orçamento enviado, Aguardando cliente, Em conserto, Pronto para retirada, Concluído e Cancelado (perdida).
- Mensagens prontas: Retorno de orçamento ("Oi {primeiro_nome}, aqui é o {responsavel} da {negocio}. O orçamento do {servico} ficou em {valor}. Posso seguir com o conserto?") e Aparelho pronto.
- 8 cartões variados, incluindo um atrasado e um sem próximo passo.

**Salão "Studio Lírio"** (dona: Camila; equipe: Mariana e Júlia)
- Etapas: Novo, Agendado, Confirmado, Atendido (concluída) e Cancelado (perdida).
- Mensagem pronta de confirmação: "Oi {primeiro_nome}! Aqui é do {negocio}. Passando para confirmar seu horário de {servico} {data} às {hora}. Posso confirmar? 💇‍♀️"
- Agendamentos espalhados na semana, com lembretes de confirmação.

## 8. Critérios de aceite

**Caso 1: assistência técnica**
1. Bruno cria "Troca de tela – iPhone 12" para o cliente **Carlos Souza**, com valor **R$ 650,00**, etapa **Orçamento enviado** e próximo passo "Retornar sobre o orçamento" na **próxima sexta às 10h**.
2. O compromisso aparece em **Próximos** e, na sexta, em **Hoje**. Depois das 10h, passa para **Atrasados**.
3. Em Hoje, Bruno toca em **WhatsApp**, revisa a mensagem com nome, serviço e valor preenchidos e abre o WhatsApp.
4. Ao voltar, conclui com **Aprovou**, muda a etapa para **Em conserto** e agenda "Avisar que está pronto" para dali a 2 dias, tudo num só **Salvar**.
5. O histórico do cartão mostra cada passo com autor e horário.

**Caso 2: salão**
1. Mariana marca para **Ana Lima** "Escova + hidratação" na **sexta às 14h**, 1h30. O lembrete de confirmação é sugerido para **quinta às 10h**.
2. Na quinta, **Confirmar horário – Ana Lima – sex. 14:00** aparece em Hoje. Mariana abre o WhatsApp com a mensagem de confirmação.
3. Conclui com **Confirmado**: o agendamento fica Confirmado na Agenda.
4. Na sexta depois das 15h30, o agendamento aparece em "Como foi?". Mariana marca **Realizado** e o cartão vai para **Atendido**.

**Gerais**
- Dono e equipe de um negócio não veem nada do outro, nem pelo Realtime.
- Contadores de Atrasados, Para hoje, Concluídos hoje e Sem próximo passo batem com as listas.
- Concluir no celular, do toque em Concluir ao Salvar com próximo passo, leva no máximo **3 toques** usando os atalhos.
- Remarcar registra datas antiga e nova no histórico.
- Etapas renomeadas e reordenadas aparecem no Quadro e nos seletores; contadores continuam corretos.
- Telefone duplicado é detectado. Busca acha "98765" e "Carlos".
- Uma mudança feita numa janela aparece na outra sem recarregar.

**Entrega:** diga como acessar `/admin`, os logins de demonstração dos dois negócios, como recriar os dados de demonstração e como reproduzir os dois casos.
