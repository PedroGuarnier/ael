# A&L Production — Agenda de Posts

App mobile (PWA) para planejar, produzir, aprovar e acompanhar os posts dos clientes. Calendário por mês e por semana, editor com foto + legenda e preview estilo Instagram, plano do mês baseado no modelo de cronograma, relatório mensal, avisos de produção, assistente por regras, radar de conteúdo com IA e sincronização entre celulares.

Sem servidor próprio: os dados ficam no aparelho (localStorage para dados, IndexedDB para fotos). A sincronização entre celulares usa um projeto gratuito do Firebase (opcional). Backup/restauração em JSON pelo próprio app.

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | O app inteiro (HTML + CSS + JS) |
| `sw.js` | Service worker: cache offline + clique na notificação |
| `manifest.webmanifest` | Permite "instalar" como app na tela inicial |
| `icon-192.png`, `icon-512.png` | Ícones do app |

Os caminhos são relativos (`./`), então funciona tanto em `usuario.github.io/repo/` quanto na raiz de um domínio.

## Publicar no GitHub Pages

1. Envie estes arquivos para a raiz da branch `main`.
2. No repositório: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `/ (root)` → Save**.
3. Em 1–2 minutos o app fica em `https://SEU-USUARIO.github.io/NOME-DO-REPO/`.
4. No Android (Chrome) aparece o botão **Instalar app** em Ajustes; no iPhone: Safari → Compartilhar → **Adicionar à Tela de Início**.

Para atualizar: troque os arquivos e faça commit. O `sw.js` usa "rede primeiro", então a próxima abertura já pega a versão nova (para forçar, mude `CACHE = 'al-posts-v3'` no `sw.js`).

## O que o app faz

- **Início** — tela inicial com a identidade A&L: saudação, contadores (hoje, a produzir, atrasados), avisos de prospecção, atalhos, próximo post, meta da semana por cliente, produção pendente e posts da semana.
- **Agenda** — mês (calendário) ou **semana** (grade clientes × dias; toque numa célula para ver o dia ou criar post; setas ‹ › nos cards movem o post de dia).
- **Hoje** — próximo post, **produção pendente** (posts dos próximos 7 dias sem foto, sem legenda ou sem aprovação), posts de hoje e da semana.
- **Editor de post** — cliente, data/hora, formato, fotos, título, legenda com contador, **pilar, objetivo, responsável, link do arquivo**, status com o fluxo do modelo (Ideia → Roteiro → Em design → Aguardando aprovação → Aprovado → Agendado → Publicado), **checklist antes de agendar** e **métricas** depois de publicado (alcance, curtidas, comentários, salvamentos, compartilhamentos, cliques).
- **Clientes › perfil** — grade do feed e três botões:
  - **Plano do mês** (modelo de cronograma): ficha do cliente, pilares com % do mês, datas e campanhas, foco por semana, **cronograma semanal base** com botão **Gerar posts do mês** (cria um post "Ideia" por linha em cada semana, só em dias futuros), fluxo de trabalho do mês, banco de ideias, checklist e erros comuns.
  - **Relatório** — por cliente ou geral, por mês: planejados, publicados, atrasados, entrega, meta, status, formatos, pilares (real vs meta), métricas somadas, top alcance, pilar que mais gerou salvamentos/contatos, sugestões para o próximo mês e as 4 perguntas de revisão. Botões: copiar texto (WhatsApp), compartilhar, salvar PDF.
  - **Radar** — IA (veja abaixo).
- **Assistente** — perguntas em português sobre o cronograma ("o que tem essa semana?", "incompletos", "atrasados", "meta", "relatório", "tendências"…). Digite `ajuda`.
- **Ajustes** — instalar, notificações, **avisos de produção**, **sincronização**, **chave da IA**, agenda do celular (.ics), nome do app, backup.

## No notebook

O mesmo endereço funciona no computador: a navegação vira uma barra lateral, a agenda mostra o mês e o dia lado a lado, a semana ganha colunas, clientes e leads viram grade, e as telas de edição abrem como janelas centradas. Atalhos de teclado: `N` novo post, `1` a `5` trocam de aba, `M` mês, `S` semana, `T` hoje, setas mudam o mês ou a semana, `Esc` fecha.

## Avisos de produção

Além do lembrete na hora do post, o app avisa (notificação + banner) quando:

- um post está a X dias de sair e ainda **falta foto ou legenda** (Ajustes › Avisar post incompleto; padrão 2 dias);
- um post está a Y dias de sair e ainda **não foi aprovado** pelo cliente (padrão 4 dias; cada cliente pode ter um prazo diferente em Clientes › Editar › Prazo de aprovação).

Valem as mesmas limitações das notificações abaixo (o app precisa estar aberto ou instalado em segundo plano).

## Login, contas e caderno pessoal

O app abre numa tela de login (e-mail e senha, pelo Firebase Authentication, sem SSO). Cada pessoa tem a sua conta, um **caderno pessoal** (botão "Meu caderno" na Início ou no avatar no topo) que só ela vê e que sincroniza entre os aparelhos dela, e pode trocar a senha em Minha conta (avatar no topo). No primeiro acesso o app sugere definir uma senha nova; dá para adiar.

Ao entrar, a pessoa já cai na equipe: o primeiro login cria a equipe e os seguintes entram sozinhos, sem convite. Depois de logado, o app funciona offline (o login fica guardado no aparelho).

Configuração no Firebase (uma vez, pelo administrador):

1. **Authentication › Métodos de login**: ative **E-mail/senha**.
2. **Authentication › Usuários › Adicionar usuário**: crie cada conta com e-mail e senha inicial. Nenhuma senha fica no código do app.
3. **Realtime Database › Regras**: cole e publique as regras abaixo.
4. Depois que todo mundo entrou pelo menos uma vez, pode desativar o método **Anônimo** em Métodos de login. A partir daí só quem tem conta acessa os dados.

```json
{
  "rules": {
    "ws": { "$code": { ".read": "auth != null", ".write": "auth != null" } },
    "team": { ".read": "auth != null", ".write": "auth != null" },
    "profiles": { "$uid": { ".read": "auth.uid == $uid", ".write": "auth.uid == $uid" } },
    "notes": { "$code": { "$uid": { ".read": "auth.uid == $uid", ".write": "auth.uid == $uid" } } }
  }
}
```

Os cadernos ficam em `notes/<código da equipe>/<usuário>` e só o próprio usuário consegue ler. O nó `team` guarda o código da equipe para os logins seguintes entrarem sozinhos.

## Sincronizar entre celulares (Firebase)

A sincronização usa o **Realtime Database** do Firebase (plano gratuito Spark é suficiente). A configuração do projeto da A&L já vem embutida no app. Com o login feito, a equipe se forma sozinha; o convite por link (Ajustes › Sincronizar) continua existindo como alternativa.

Com a sincronização ligada, as chaves da IA, do workspace e do Google Maps também são compartilhadas pelo mesmo canal: quem salvar uma chave num celular, o outro recebe sozinho (elas ficam no nó `keys` do mesmo workspace, protegido pelas mesmas regras).

Como funciona: cada post, cliente e lead tem um carimbo de hora; a edição mais recente vence. As fotos sobem uma vez e são baixadas no outro aparelho quando faltam. Sem internet o app continua funcionando e envia as mudanças quando voltar. Notificações e ajustes são por aparelho (não sincronizam).

## Mapa da prospecção (OpenStreetMap, sem chave)

O passo **1 · Varrer bairros no mapa** consulta o OpenStreetMap (serviço Overpass), que é gratuito e não precisa de chave, cartão nem console do Google. Para cada bairro, o app busca os negócios dos segmentos escolhidos num raio de 2,5 km do centro do bairro, com nome, endereço, telefone, site e Instagram quando cadastrados. Os candidatos são ordenados por sinais de oportunidade (sem site, sem Instagram cadastrado, sem telefone). Depois:

- **2 · Avaliar com IA** — manda até 40 candidatos para a IA, que confere o Instagram dos mais promissores na web e escolhe de 8 a 12 leads com sinais e abordagem.
- **Adicionar todos** — vira leads sem IA, com prioridade pelos sinais do mapa.

Limites: a cobertura do OpenStreetMap no Rio é boa para restaurantes, bares, lojas, salões e academias, e irregular para clínicas e consultórios pequenos. Não há avaliações. Se o serviço estiver ocupado, o app tenta um segundo servidor e pede para repetir em 1 minuto.

Opcional: se um dia quiserem a cobertura do Google Maps (com notas e avaliações), cole uma chave da API do Google Maps em Ajustes › Mapa da prospecção e a varredura passa a usar o Google. Exige projeto no Google Cloud com cartão, Places API (New) e Maps JavaScript API ativas e chave restrita ao site `https://pedroguarnier.github.io/*`.

## Radar de conteúdo (IA)

Usa a API da Anthropic (Claude) direto do celular, com a sua chave. Em **Ajustes → Radar de conteúdo**, cole a chave (`sk-ant-…`, criada em console.anthropic.com → API Keys). A chave fica só no aparelho e não entra no backup.

No perfil do cliente, **Radar**:

1. **Estudar perfil** — a IA lê ficha, pilares, posts e métricas e escreve um resumo (posicionamento, público, o que funciona, lacunas, direção do mês). Fica salvo no cliente.
2. **Buscar tendências** — a IA pesquisa na web o que está em alta no nicho no Brasil (formatos, trends, datas comemorativas dos próximos 30 dias) e propõe 6 ideias adaptadas ao cliente, cada uma com formato, pilar, gancho, legenda pronta, justificativa e fonte. Cada ideia tem **Criar post** (abre o editor preenchido) e **Guardar ideia** (banco de ideias).

Custo: o app usa o modelo Sonnet 5.5. Cada busca de tendências fica na faixa de US$ 0,30 a 0,50 (até 8 buscas na web); o estudo de perfil custa cerca de US$ 0,03. Para 3 clientes com um estudo e duas buscas por mês cada, dá uns US$ 2 a 3 por mês. A conta da Anthropic precisa ter crédito.

## Notificações — o que funciona onde

| Situação | Android (Chrome, instalado) | iPhone (instalado via Safari) | Navegador comum |
|---|---|---|---|
| App aberto | ✅ notificação + banner + som | ✅ | ✅ |
| App em segundo plano / tela bloqueada | ✅ enquanto o sistema mantiver o app vivo | ⚠️ não confiável | ❌ |
| App fechado | ❌ | ❌ | ❌ |

Nenhum app web consegue disparar notificação agendada com o app **fechado** sem um servidor de push. Por isso o app exporta `.ics` (Ajustes → Agenda do celular) e cada post tem botão **Google Agenda**.

## Publicar no Instagram

O Instagram só permite agendamento externo com conta comercial + Meta Graph API (precisa de backend e aprovação). O app deixa o post "pré-programado": na hora, o botão **Publicar agora** copia a legenda, compartilha as fotos para o Instagram pelo menu do celular e abre o app. Depois é só marcar como publicado e anotar as métricas.

## Dados e privacidade

- Sem sincronização: tudo fica no aparelho. Faça backup em Ajustes.
- Com sincronização: posts, clientes, planos, relatórios e fotos ficam no seu projeto Firebase (conta Google de vocês). Quem tiver o código da equipe e a configuração acessa tudo; troque o código se vazar (desligue e ligue de novo com outro código nos dois celulares).
- A chave da IA só é enviada para a API da Anthropic.
