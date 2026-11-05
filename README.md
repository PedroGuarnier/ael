# Agenda de Posts — A&L

App mobile (PWA) para planejar posts dos clientes, com calendário, editor com foto + legenda e preview estilo Instagram, assistente de cronograma por regras, aba de clientes com logo e notificações no celular.

Sem backend: tudo fica no aparelho (localStorage para dados, IndexedDB para fotos/logos). Backup/restauração em JSON pelo próprio app.

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | O app inteiro (HTML + CSS + JS) |
| `sw.js` | Service worker: cache offline + clique na notificação |
| `manifest.webmanifest` | Permite "instalar" como app na tela inicial |
| `icon-192.png`, `icon-512.png` | Ícones do app |

Os caminhos são relativos (`./`), então funciona tanto em `usuario.github.io/repo/` quanto na raiz de um domínio.

## Publicar no GitHub Pages

1. Crie um repositório (ex.: `agenda-posts`) e envie estes 5 arquivos para a raiz da branch `main`.
2. No repositório: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `/ (root)` → Save**.
3. Em 1–2 minutos o app fica em `https://SEU-USUARIO.github.io/agenda-posts/`.
4. Mande o link para a Luana. No Android (Chrome) aparece o botão **Instalar app** em Ajustes; no iPhone: Safari → Compartilhar → **Adicionar à Tela de Início**.

Para atualizar: troque os arquivos e faça commit. Como o `sw.js` usa "rede primeiro", a próxima abertura já pega a versão nova (se quiser forçar, mude `CACHE = 'al-posts-v3'` no `sw.js`).

## Notificações — o que funciona onde

| Situação | Android (Chrome, instalado) | iPhone (instalado via Safari) | Navegador comum |
|---|---|---|---|
| App aberto | ✅ notificação + banner + som | ✅ | ✅ |
| App em segundo plano / tela bloqueada | ✅ enquanto o sistema mantiver o app vivo | ⚠️ não confiável | ❌ |
| App fechado | ❌ | ❌ | ❌ |

Nenhum app web consegue disparar notificação agendada com o app **fechado** sem um servidor de push. Por isso o app exporta `.ics` (Ajustes → Agenda do celular) e cada post tem botão **Google Agenda** — o calendário do celular dá o alarme em qualquer situação. Se quiser alerta por WhatsApp, o próximo passo é um script no servidor lendo o backup JSON.

## Publicar no Instagram

O Instagram só permite agendamento externo com conta comercial + Meta Graph API (precisa de backend e aprovação). O app deixa o post "pré-programado": na hora, o botão **Publicar agora** copia a legenda, compartilha as fotos direto para o Instagram pelo menu de compartilhar do celular e abre o app. Depois é só marcar como postado.

## Assistente (motor de regras)

Funciona offline, sem IA externa. Entende perguntas em português sobre o cronograma: período (hoje, amanhã, semana, dia 12, sexta, novembro…), cliente, formato (reels, stories, carrossel, feed) e intenção (listar, contar, próximo, atrasados, incompletos, dias livres, meta, melhor horário, ideias, agendar). Digite `ajuda` no chat para ver os exemplos. Para trocar por uma IA de verdade, o ponto de entrada é a função `answer(q)` no `index.html`.
