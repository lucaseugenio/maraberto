G U I A D O A P P 

# 🌊 Mar Aberto — Financeiro da família 

Um webapp de um único arquivo ( **`index.html`** ) para acompanhar as finanças da família: lançamentos manuais, cartões com faturas, contas de todos os membros (você, esposa e filha), metas, relatórios e projeção futura — “quanto terei e quanto estarei devendo em tal data”. 

- ✅ HTML + CSS + JS puros 🚫 sem build e sem bibliotecas 🔓 sem login 

⚓ logo embutido 

0 1 U S O IM E D IAT O 

## Como usar agora mesmo 

Não existe instalação: o aplicativo vive inteiro dentro de um arquivo. Isso significa que você pode testar antes de decidir qualquer coisa, e que uma cópia de segurança é só uma cópia do arquivo. 

- 1 Abra o **`index.html`** no navegador (duplo clique). Pronto — sem configurar nada, os dados ficam salvos no próprio navegador (modo local). 

- 2 Para ver o app preenchido, vá em ⚙ Configurações **→** 💾 Backup **→** ✨ Dados de exemplo. 

💡 O modo local guarda os dados no **`localStorage`** do navegador: funciona offline, mas fica só naquele navegador e naquele dispositivo. Para acessar de qualquer lugar ou dividir com a família, siga as seções 02 e 03. 

0 2 PU B LIC AÇ ÃO 

## Colocar no GitHub (acessar de qualquer lugar) 

O GitHub Pages publica o arquivo de graça e cria um endereço fixo que abre em computador, celular ou tablet. O processo é feito uma única vez. 

1 Crie um repositório (ex.: **`mar-aberto`** ) e suba o **`index.html`** . 

- 2 Em Settings **→** Pages **→** Source: Deploy from a branch **→** main / (root) → Save. 3 Em alguns minutos o app estará em **`https://seuusuario.github.io/maraberto/`** . 

🔄 Ao atualizar o **`index.html`** no GitHub, o navegador pode mostrar a versão antiga por alguns minutos (cache). Faça Ctrl+F5 (recarregar sem cache) se notar algo velho. 

0 3 B AN C O D E D AD O S 

## Conectar o Supabase (lançamentos em família) 

O Supabase é um banco de dados gratuito que faz o papel de “servidor”: os lançamentos passam a ficar lá, e cada dispositivo lê e escreve no mesmo lugar. A conta gratuita serve perfeitamente para este uso. 

- 1 Crie uma conta em supabase.com e um novo projeto (o gratuito serve). 

- 2 No menu lateral, abra SQL Editor, cole o SQL abaixo e clique em Run — ele cria as 4 tabelas e libera o acesso anônimo. O app não tem login, conforme combinado; um eventual vazamento não expõe nada importante. 

SQ L D E C R IAÇ ÃO D AS TAB E LAS 

```
create extension if not exists pgcrypto;
```

```
create table if not exists contas (
```

```
  id uuid primary key, nome text not null,
```

- `banco text default 'outro', tipo text default 'corrente', titular text default '',` 

```
  "saldoInicial" numeric default 0, meta numeric, cor text default '#5C93A6',
  observacoes text default '', "criadoEm" timestamptz default now());
create table if not exists cartoes (
```

```
  id uuid primary key, nome text not null,
```

```
  bandeira text default 'visa', banco text default 'outro', titular text default
'',
```

- `"diaFechamento" int default 20, "diaVencimento" int default 5, limite numeric, cor text default '#5C93A6', "criadoEm" timestamptz default now()); create table if not exists categorias (` 

`id uuid primary key, nome text not null, tipo text default 'despesa', icone text default '` 🏷 `', cor text default '#5C93A6', sistema boolean default false,` 

- `"criadoEm" timestamptz default now());` 

```
create table if not exists lancamentos (
```

- `id uuid primary key, data date not null, descricao text default '', valor numeric default 0, tipo text default 'despesa',` 

- `"contaId" uuid, "cartaoId" uuid, "categoriaId" uuid, titular text default '', pago boolean default true, observacoes text default '', grupo text, "criadoEm" timestamptz default now());` 

```
alter table contas enable row level security;
alter table cartoes enable row level security;
```

```
alter table categorias enable row level security;
alter table lancamentos enable row level security;
create policy "anon total contas" on contas for all to anon using (true) with
check (true);
create policy "anon total cartoes" on cartoes for all to anon using (true) with
check (true);
create policy "anon total categorias" on categorias for all to anon using (true)
with check (true);
create policy "anon total lancamentos" on lancamentos for all to anon using
(true) with check (true);
```

(Esse mesmo SQL está dentro do app: ⚙ Configurações **_→_** 🗄 Armazenamento **_→_** Ver SQL de criação das tabelas **_→_** Copiar.) 

- 3 Em Project Settings **→** API, copie a Project URL e a anon public key. 

- 4 No app: ⚙ Configurações **→** 🗄 Armazenamento → cole os dois → Salvar e conectar. 

- Se você já tinha dados no navegador, eles são enviados automaticamente ao Supabase na primeira conexão. 

De lá em diante, todo lançamento salvo vai para o Supabase e continua com uma cópia local — se a internet cair, você não perde nada: o app avisa e sincroniza depois, quando você salvar de novo. 

- 0 4 R E G R AS 

## Como o dinheiro é calculado 

As quatro regras abaixo são o coração do app — vale conferir com calma para confiar nos números que ele mostra. 

### 💰 Conta 

Saldo inicial + receitas − despesas − pagamentos de fatura, tudo pela data do lançamento. A data manda: lançamento de amanhã não afeta o saldo de hoje. 

### 💳 Cartão 

Compras no cartão viram dívida imediatamente; o dinheiro só sai da conta quando você registra o “Pagar fatura”. O app agrupa as compras por ciclo usando o dia de fechamento e mostra o vencimento e os dias restantes. 

### 🔮 Projeção 

Escolheu uma data? O app soma todos os lançamentos até lá (pagos ou agendados) e mostra: dinheiro em contas, dívida nos cartões e o saldo líquido naquele dia — além da curva do saldo e da lista do que acontece até lá, inclusive faturas que vão vencer. 

### 🎯 Meta nas contas 

Opcional: defina um valor-alvo na conta e veja a barra de progresso com o % atual, ou 🎉 quando a meta é atingida. Bom para reservas (viagem, colchão de segurança). 

0 5 D AD O S 

## Estrutura dos dados 

Quatro tabelas guardam tudo o que o app conhece — as mesmas do SQL e as mesmas do backup. 

|Tabela|Guarda|
|---|---|
|contas|bancos, tipo, titular, saldo inicial,metae cor|
|cartoes|bandeira (Visa, Mastercard, Elo, Amex, Hipercard…), banco, dia de<br>fechamento e vencimento, limite|
|categorias|nome, tipo (despesa/receita), emoji e cor — 21 básicas já vêm<br>prontas e são editáveis|
|lancamentos|data, descrição, valor, tipo, conta/cartão, categoria, membro da<br>família, status e observações|



💾 Backup: ⚙ Configurações → 💾 Backup → baixa um JSON com tudo e permite restaurar depois (incl. enviar para o Supabase). 

0 6 ID E N T ID AD E 

## O logo e o visual 

O tema é pastel e lavado. O logo oficial (âncora náutica) já vem embutido no **`index.html`** — aparece no topo e na aba do navegador (favicon), igual em qualquer dispositivo. Se um dia quiser trocar por outra arte sem editar o arquivo: ⚙ Configurações **→** 🖼 Logotipo **→** Enviar logo (fica salvo no navegador). Para voltar ao logo oficial, use usar logo padrão. 

Mar Aberto é o projeto; Financeiro é a frente — a inscrição abaixo do nome saiu, e as próximas frentes entram como abas ao lado de “Financeiro”. 

0 7 S E G U IM O S 

## Próximas conversas (roadmap) 

- Abas futuras do Mar Aberto — o menu já reserva o lugar: “+ novas abas em breve”; Ajustes finos de design que surgirem no uso do dia a dia; 

- Editar/excluir em série, categorias recorrentes, gráficos comparativos ano a ano — é só pedir. 

O app foi feito para crescer junto com a família: começa pelo Financeiro e ganha frentes novas conforme fizer sentido. Qualquer ajuste que aparecer no uso — um campo a mais, uma categoria que falta, uma conta que mudou de banco — é uma conversa curta. 

- 🧭 Recapitulando — do zero ao “em família” 

1. Baixe o **`index.html`** e abra com um duplo clique — modo local, nada para configurar.  2. Crie suas contas e cartões, ou veja o app preenchido com ✨ Dados de exemplo.  3. Registre os lançamentos do dia a dia, pagos ou agendados — a projeção cuida do futuro.  4. Publique no GitHub Pages para acessar de qualquer dispositivo.  5. Conecte o Supabase quando quiser que a família compartilhe os mesmos dados.  6. Baixe um backup JSON de vez em quando, sobretudo antes de grandes mudanças. 

> ⚓ Mar Aberto · guia do app 

um único arquivo · seus dados, seus lugares 

