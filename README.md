# Lista de Merenda

Projeto simples em HTML/JS puro (sem build) para montar um pedido de merenda,
fechá-lo e salvar em um banco de dados gratuito (Supabase). Pode ser hospedado
de graça no GitHub Pages.

## 1. Criar o banco de dados (Supabase, gratuito)

1. Crie uma conta em https://supabase.com e um novo projeto (plano Free).
2. No painel do projeto, vá em **SQL Editor** e rode:

   ```sql
   create table merenda_itens (
     id bigint generated always as identity primary key,
     nome_pessoa text not null,
     item text not null,
     quantidade int not null default 1,
     status text not null default 'aberto',
     criado_em timestamptz not null default now(),
     fechado_em timestamptz
   );

   alter table merenda_itens enable row level security;

   create policy "Permitir insercao publica"
     on merenda_itens for insert
     with check (true);

   create policy "Permitir leitura publica"
     on merenda_itens for select
     using (true);

   create policy "Permitir atualizacao publica"
     on merenda_itens for update
     using (true);

   create policy "Permitir delete publica"
     on merenda_itens for delete
     using (true);
   ```

   Isso cria a tabela e libera inserir/ler/atualizar/remover itens
   publicamente (ok para uso pessoal simples; não coloque dados sensíveis).

3. Ative o **Realtime** na tabela, para que a lista de itens atualize
   sozinha para todo mundo assim que alguém adicionar, remover ou fechar
   um item:
   - Vá em **Database > Replication** (ou **Table Editor**, ícone de
     Realtime ao lado da tabela) e ative a replicação para
     `merenda_itens`.
   - Ou rode no SQL Editor:
     ```sql
     alter publication supabase_realtime add table merenda_itens;
     ```

4. Vá em **Settings > API** e copie a **Project URL** e a chave **anon public**.

> Se você criou a tabela `merenda_pedidos` de uma versão anterior deste
> projeto, pode apagá-la — ela não é mais usada.

## 2. Configurar o projeto

1. Abra o arquivo `config.js` e preencha:

   ```js
   window.SUPABASE_URL = "https://SEU-PROJETO.supabase.co";
   window.SUPABASE_ANON_KEY = "SUA_CHAVE_ANON_PUBLICA";
   ```

   A chave "anon" é feita para ser pública (o acesso é controlado pelas
   *policies* de RLS acima), então é seguro publicá-la no GitHub.

2. Teste localmente: abra `index.html` no navegador (duplo clique) ou sirva
   a pasta com qualquer servidor estático.

## 3. Publicar no GitHub Pages

```bash
git init
git add index.html config.js config.example.js README.md
git commit -m "Lista de merenda"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/SEU-REPO.git
git push -u origin main
```

Depois, no GitHub: **Settings > Pages > Source: branch `main`, pasta `/`**.
O site ficará disponível em `https://SEU-USUARIO.github.io/SEU-REPO/`.

> Nota: o `.gitignore` deste projeto ignora `config.js` por padrão (para
> evitar commit acidental antes de configurar). Como a chave anon é
> pública por design, remova a linha `config.js` do `.gitignore` (ou use
> `git add -f config.js`) para que ele seja publicado junto no Pages —
> sem isso o site no ar não vai conseguir salvar os pedidos.

## Como funciona

- Cada pessoa preenche o próprio nome uma vez (fica salvo no navegador).
- Ao adicionar um item, ele já é salvo no Supabase e aparece na hora para
  todo mundo que estiver com a página aberta (via Supabase Realtime).
- Qualquer um pode remover um item da lista ativa antes de fechar o pedido.
- Clique em **Fechar Pedido** para arquivar todos os itens da lista atual;
  eles saem da lista ativa e passam a aparecer no histórico abaixo.
