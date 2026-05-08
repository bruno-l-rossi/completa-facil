# Completa Fácil — Álbum Copa 2026

Aplicativo web para gerenciar o álbum Panini da Copa do Mundo 2026 e facilitar trocas de figurinhas entre colecionadores em São Carlos, SP.

---

## O que é

Um Progressive Web App (PWA) desenvolvido em HTML, CSS e JavaScript puro — sem frameworks, sem build step. Funciona direto no browser de qualquer dispositivo, com design otimizado para mobile.

**Funcionalidades principais:**

- Cadastro e gerenciamento das 980 figurinhas do álbum oficial Panini
- Sistema de busca por seleção, jogador ou código (ex: `BRA9`, `FWC10`)
- Filtros de coleção: Incompletas, Repetidas e Coletadas
- Configuração de preferências de troca (raridade, o que oferece, o que precisa)
- Matches automáticos entre usuários com base na coleção e preferências
- Sistema de sessões de encontro fixas semanais com confirmação de presença
- Sugestão de horários de troca e confirmação bilateral entre pares

---

## Tecnologias

- **Frontend:** HTML5, CSS3, JavaScript (ES2020+) — arquivo único `index.html`
- **Backend/Banco:** [Supabase](https://supabase.com) (PostgreSQL + REST API)
- **Hospedagem:** [Cloudflare Pages](https://pages.cloudflare.com)
- **Fontes:** Google Fonts (Bebas Neue + Nunito)

---

## Estrutura do banco de dados (Supabase)

```sql
-- Usuários
create table usuarios (
  id uuid default gen_random_uuid() primary key,
  username text unique not null,
  senha_hash text not null,
  criado_em timestamptz default now()
);

-- Coleções e configurações de troca
create table colecoes (
  id uuid default gen_random_uuid() primary key,
  usuario_id uuid references usuarios(id) on delete cascade,
  collection jsonb default '{}' not null,  -- {figurinhas: {}, trades: {}}
  atualizado_em timestamptz default now()
);

-- Confirmações de encontro entre pares
create table confirmacoes (
  id uuid default gen_random_uuid() primary key,
  usuario_a_id uuid references usuarios(id) on delete cascade,
  usuario_b_id uuid references usuarios(id) on delete cascade,
  sessao_key text not null,
  confirmado_a boolean default false,
  confirmado_b boolean default false,
  criado_em timestamptz default now(),
  unique(usuario_a_id, usuario_b_id)
);
```

---

## Sessões de troca semanais

As trocas acontecem em dois pontos fixos em São Carlos:

| Dia | Horários | Locais |
|-----|----------|--------|
| Terça | 17h–18h | Shopping Iguatemi e Praça XV |
| Quinta | 17h–18h | Shopping Iguatemi e Praça XV |
| Sexta | 17h–18h | Shopping Iguatemi e Praça XV |
| Sábado | 10h–11h, 15h–16h, 16h–17h, 17h–18h | Shopping Iguatemi e Praça XV |
| Domingo | 10h–11h, 15h–16h, 16h–17h, 17h–18h | Shopping Iguatemi e Praça XV |

> ⚠️ **Menores de idade devem comparecer acompanhados de um responsável.**

---

## Como rodar localmente

Não há dependências nem build necessário. Basta servir o arquivo HTML:

```bash
# Python
python3 -m http.server 3000

# Node
npx serve .
```

Abra `http://localhost:3000` no browser.

> **Atenção:** não abra o arquivo diretamente (`file://`) — o browser bloqueia requisições ao Supabase por segurança.

---

## Deploy

O app é hospedado no Cloudflare Pages com deploy automático a cada push na branch `main`.

Para atualizar:

```bash
git add index.html
git commit -m "descrição da atualização"
git push
```

---

## Configuração do Supabase (keep-alive)

O plano gratuito do Supabase pausa projetos após 7 dias sem uso. Para evitar isso, um cron no [cron-job.org](https://cron-job.org) chama o endpoint de keep-alive a cada 5 dias:

```
URL: https://SEU-PROJETO.supabase.co/rest/v1/rpc/keep_alive
Header: apikey: SUA_ANON_KEY
Intervalo: a cada 5 dias
```

---

## Figurinhas

O álbum contém **980 figurinhas** no total:

- **20 especiais** (FWC1–FWC19 + Logo Panini) — todas brilhantes
- **48 seleções × 20 figurinhas** = 960

Cada seleção contém: Logo FOIL (brilhante), 11 jogadores, Foto do Time, 7 jogadores.

---

## Segurança

- Senhas armazenadas com `btoa()` — hash simples adequado para o contexto
- Row Level Security (RLS) desabilitada na tabela `confirmacoes` (dados públicos por natureza)
- Nenhum dado sensível (email, telefone, localização) é coletado
- Comunicação entre usuários é feita exclusivamente via encontros presenciais em locais públicos

---

## Contribuindo

Projeto desenvolvido para uso local em São Carlos, SP. Para sugestões ou correções, abra uma issue neste repositório.
