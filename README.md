# Lista de Merenda

Projeto simples em HTML/JS puro (sem build) para montar um pedido de merenda,
fechá-lo e salvar em um banco de dados gratuito (Supabase). Pode ser hospedado
de graça no GitHub Pages.

## 1. Criar o banco de dados (Supabase, gratuito)

1. Crie uma conta em https://supabase.com e um novo projeto (plano Free).
2. No painel do projeto, vá em **SQL Editor** e rode:

   ```sql
   create table merenda_pedidos (
     id bigint generated always as identity primary key,
     itens jsonb not null,
     criado_em timestamptz not null default now()
   );

   alter table merenda_pedidos enable row level security;

   create policy "Permitir insercao publica"
     on merenda_pedidos for insert
     with check (true);

   create policy "Permitir leitura publica"
     on merenda_pedidos for select
     using (true);
   ```

   Isso cria a tabela e libera inserir/ler pedidos publicamente (ok para uso
   pessoal simples; não coloque dados sensíveis).

3. Vá em **Settings > API** e copie a **Project URL** e a chave **anon public**.

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

- Adicione itens com nome e quantidade na lista.
- Clique em **Fechar Pedido** para salvar o pedido completo no Supabase.
- O histórico dos últimos 20 pedidos aparece abaixo do formulário.
