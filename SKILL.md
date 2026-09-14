---
name: avaliador-checkin-relatorio-c2g
description: 'Skill oficial da agência C2G (2Gather) para avaliar OU gerar check-ins semanais e relatórios mensais de tráfego pago (Meta Ads/Google Ads) que o Well envia a clientes, e para montar os blocos internos usados no formulário da agência. Se o Well mandar um PDF de dados brutos (relatório/dashboard de métricas, sem texto corrido): identifica automaticamente se é check-in semanal ou relatório mensal, busca no Google Drive o histórico e o padrão de comunicação daquele cliente específico, e gera só um rascunho do check-in/relatório pra revisão (sem avaliação, sem blocos ainda). Se o Well colar/reenviar um texto (já escrito, ou o rascunho revisado): verifica se responde às 6 perguntas obrigatórias do processo da C2G (problema da conta, ação tomada, resultado do teste, resultado do período vs. anterior e por quê, próximos passos, pedido ao cliente), retorna "Status: Aprovado" ou "Status: Reprovado" com correções objetivas, e SÓ monta os 4 blocos internos (problema/ação, resultado/estado atual, próximos passos, pedido ao cliente) se o status for Aprovado — se reprovado, para nas correções e espera o Well reenviar ajustado. Em toda avaliação e em todo rascunho gerado, também checa vícios de escrita de IA e proibição de travessão (regras da skill copywriter-supremo aplicadas ao contexto de check-in). Também processa em lote: se o Well pedir pra gerar/processar todos os check-ins de uma vez (ex.: "gera todos os checkins", "processa a entrada"), varre a pasta "Entrada de relatórios" do Drive (inbox semanal e efêmera onde ele sobe os PDFs de todos os clientes toda segunda-feira e apaga no mesmo dia) e gera um rascunho por cliente encontrado lá. Use SEMPRE que o Well colar um check-in/relatório pra conferir, avaliar, validar, revisar ou aprovar; anexar um PDF de relatório de cliente pedindo para gerar/montar/rascunhar o check-in ou relatório; pedir pra processar/gerar tudo de uma vez ou mencionar a pasta "Entrada de relatórios"; pedir para "montar os blocos", "gerar os blocos internos"; ou mencionar check-in semanal / relatório mensal — mesmo sem usar a palavra "skill", "avaliar" ou "gerar" explicitamente.'
---

# Avaliador de Check-in e Relatório Mensal — C2G

Skill da agência C2G (2Gather) para conferir se check-ins semanais e relatórios mensais enviados a clientes de tráfego pago cobrem o essencial do processo interno da agência, e para extrair o conteúdo em blocos padronizados para o formulário interno.

## Fonte de verdade — não resumir, consultar quando precisar de detalhe

Este SKILL.md traz o resumo operacional. O conteúdo original, na íntegra, está em `references/`:

- `references/instrucoes-originais-well.md` — as instruções exatas que o Well escreveu para esta skill (check-ins e relatórios mensais). Em caso de qualquer dúvida ou aparente conflito, **este arquivo prevalece** sobre o resumo abaixo.
- `references/sop-checkin-semanal-c2g.md` — o SOP interno da C2G sobre check-in semanal (PDF "Direcionamento sobre os checkins"), na íntegra.
- `references/transcricao-aula-checkin-c2g.md` — a transcrição completa da aula de treinamento interno sobre check-in, com o racional completo (dado vs. informação, os 3 objetivos, a "receita de bolo", boas práticas, o que é proibido, e dois exemplos comentados: um check-in ruim analisado linha a linha e um check-in bom, incluindo como o Well estrutura a resposta às 6 perguntas antes de escrever o check-in).
- `references/vicios-de-ia-checklist.md` — checklist completo de vícios de escrita de IA e proibição de travessão, adaptado da skill `copywriter-supremo` pro contexto de check-in. O SKILL.md traz o resumo (seção "Revisão de vícios de IA e travessão" abaixo); use o arquivo quando precisar checar um item específico com mais detalhe.
- `references/contas-meta-ads.md` — como descobrir o `act_<ID>` de um cliente via `/me/adaccounts` quando precisar (a skill não guarda uma lista fixa de clientes, ver arquivo pro porquê); ver seção "Acesso à API do Meta Ads" abaixo pro protocolo completo (e a regra de segurança do token).

Consulte os arquivos de referência sempre que: o Well pedir para gerar um check-in do zero (não só avaliar um já pronto); pedir para explicar o "porquê" de alguma regra; pedir exemplos completos de check-in bom/ruim; ou o caso em mãos for uma borda que o resumo abaixo não cobre claramente.

## Acesso à API do Meta Ads (Graph API) — dados extras além do PDF

O gestor normalmente mantém um PDF chamado `Token do Meta Ads.pdf` na raiz da pasta configurada em `config.local.md` (ver "Onde ficam os arquivos dos clientes" abaixo), com um access token de longa duração do Meta Ads. Se não achar esse arquivo ou não souber onde ele está, pergunte ao gestor — e, se ele disser que ainda não tem um token, guie-o pelo passo a passo abaixo. Use o token quando o PDF de métricas que ele mandou não tiver informação suficiente pra escrever um bom check-in (ex.: precisa saber se uma campanha específica está ativa ou pausada agora, o texto exato de um anúncio, criativos disponíveis, estrutura de campanhas de uma conta) — nesses casos, consulte a Graph API diretamente em vez de pedir pro gestor mandar mais dado.

### Se o gestor ainda não tiver um token — como gerar um

O ideal é um token de **Usuário do Sistema** (System User) do Business Manager, porque esse tipo não expira em 60 dias como um token de usuário pessoal — só é revogado manualmente. Passo a passo:

1. **Crie um App no Meta for Developers** (se a agência ainda não tiver um): acesse [developers.facebook.com/apps](https://developers.facebook.com/apps), "Criar app" → tipo **Negócios** → dê um nome (ex.: "C2G Ads Access") → conclua a criação. No painel do app, adicione o produto **Marketing API**.
2. **Crie (ou reaproveite) um Usuário do Sistema no Business Manager**: acesse [business.facebook.com/settings](https://business.facebook.com/settings) → **Usuários** → **Usuários do sistema** → "Adicionar" → nomeie (ex.: "Claude Code - Check-ins") e defina a função como **Admin**.
3. **Atribua as contas de anúncio a esse usuário do sistema**: ainda na tela do usuário do sistema, clique em "Atribuir ativos" → aba **Contas de anúncios** → marque todas as contas de clientes que precisam ser consultadas (ou todas as contas da agência, se o gestor tiver acesso a todas) → conceda controle total.
4. **Gere o token**: na mesma tela do usuário do sistema, clique em "Gerar novo token" → selecione o App criado no passo 1 → marque as permissões: `ads_read`, `ads_management`, `business_management`, `read_insights` → em **Expiração do token**, escolha **Nunca expira** (essa opção só existe pra usuário do sistema, não pra token de usuário pessoal) → gere.
5. **Copie o token gerado imediatamente** — o Meta só mostra o valor uma vez; se perder, precisa gerar outro.

### Onde colocar o token pra a skill ter acesso

1. Cole o token (só o valor, nada mais) num arquivo chamado exatamente `Token do Meta Ads.pdf` (pode ser um PDF exportado de um Google Doc com só o token colado, ou qualquer PDF simples de texto).
2. Salve esse arquivo na **raiz** da mesma pasta do Drive configurada em `config.local.md` (a mesma que tem `Checkin semanal/`, `Relatório mensal/`, etc.) — não dentro de uma subpasta.
3. Trate esse arquivo como senha: não compartilhe o link dele fora da equipe, e se o token vazar ou for comprometido, revogue-o em Business Settings → Usuários do sistema → (o usuário) → "..." → Excluir token, e gere um novo.

**Regra de segurança inegociável — o token nunca é persistido nem exibido:**
- **Nunca** escreva o token num arquivo em disco (nem temporário, nem no scratchpad) — isso é bloqueado pelo classificador de segurança do Claude Code ("Credential Materialization") e, mesmo que não fosse, é uma prática ruim.
- **Nunca** imprima o valor do token na resposta pro usuário, em bloco de código, ou em qualquer texto que fique visível no chat.
- Cada vez que precisar usar o token: leia o PDF `Token do Meta Ads.pdf` com a ferramenta de leitura de arquivo, e use o valor **na mesma chamada de Bash**, inline (`TOKEN='...' && curl ...`), sem nunca gravar em arquivo. Não reutilize entre chamadas salvando em variável de ambiente persistente — leia e use, sempre no mesmo golpe.
- Se uma tentativa de persistir o token em arquivo for bloqueada pelo classificador de auto mode, **não tente contornar** — use a abordagem inline, ou pare e avise o gestor.

**Como consultar:**
- Endpoint base: `https://graph.facebook.com/v21.0/<node>?fields=<campos>&access_token=<token>`
- Pra descobrir/confirmar o ID da conta de um cliente: `/me/adaccounts?fields=id,name,account_status`, achando pelo nome mais próximo (ver `references/contas-meta-ads.md` pro protocolo completo de como escolher entre contas parecidas/duplicadas).
- Pra campanhas de uma conta: `/act_<ID>/campaigns?fields=name,status,effective_status`
- Pra anúncios/criativos: `/act_<ID>/ads?fields=name,status,creative{title,body,image_url}`
- **Detalhe técnico importante:** quando um campo usa `{}` (campos aninhados, ex.: `creative{title,body}`) ou o parâmetro `time_range` (que é um JSON, ex.: `{"since":"2026-09-07","until":"2026-09-13"}`), **não** monte a URL manualmente com esses caracteres direto no comando `curl` — o shell quebra as chaves. Use `curl -sG "<endpoint>" --data-urlencode "fields=creative{title,body}" --data-urlencode 'time_range={"since":"...","until":"..."}' --data-urlencode "access_token=${TOKEN}"`, que faz o URL-encode certo.

**Diagnosticar uma queda, anomalia ou "o que foi feito essa semana" (P2) — SEMPRE prefira o histórico de alterações da conta, não adivinhar por `updated_time`:**

1. **Primeira escolha, sempre: o log de atividades da conta**, pedindo **sempre** o campo `actor_name` junto (é essencial, ver item 2): `/act_<ID>/activities?fields=event_type,event_time,translated_event_type,extra_data,object_name,actor_name&since=<data>&until=<data>`. Isso mostra o histórico real de mudanças: quem pausou o quê, quando, e **por quê** — inclusive se foi uma automação (regra) ou uma ação manual. É muito mais confiável que adivinhar por `updated_time` de `/campaigns`, que só diz *quando* algo mudou, não *o que* mudou, *quem* mudou, nem *por quê*. Use isso toda vez que precisar entender uma anomalia ou encontrar o P2 de um cliente.
2. **Sempre filtre por `actor_name`.** Se vier `"Meta"`, é o sistema (automação/regra/revisão de anúncio) — **nunca é P2**, mesmo sem `rule_info` preenchido. Só conta como ação do Well um evento com `actor_name` de uma pessoa (ex.: "Wellington Filho") ou de outro membro da equipe. Preste atenção também no `event_type`: `update_campaign_run_status` com `rule_info` preenchido = automação/regra agendada (ex.: "Desativar campanhas de captação na sexta 18h"), nunca reporte como P2 mesmo que o `actor_name` pareça humano.
3. **Nunca assuma a direção da mudança pelo nome do evento — sempre leia `old_value` e `new_value` dentro de `extra_data`.** Um evento chamado "update_ad_set_run_status" pode ser tanto uma ativação quanto uma pausa; só o `old_value`/`new_value` (ex.: `"Ativo" -> "Inativo"`) diz qual foi. Já aconteceu de uma sequência de eventos parecer "criou e subiu criativo novo" quando na verdade terminou em "criou, testou brevemente, e pausou/excluiu tudo no mesmo dia" — sem checar o valor final de cada mudança, a skill teria reportado uma ação que não corresponde ao estado real. Reconstrua a sequência cronologicamente e veja o **estado final**, não só o que foi tocado.
4. `ad_account_update_status` (com `actor_name: "Meta"`) = **mudança no status de pagamento/saúde da conta** (ex.: de "Ativa" pra "Pagamento necessário") — isso é crítico e sempre deve ir pro check-in como ponto de atenção urgente, não pode passar batido, mesmo sendo um evento do sistema. Se aparecer, confirme com `/act_<ID>?fields=account_status,disable_reason,funding_source_details` pra ver o status atual e a forma de pagamento cadastrada, e avise o Well claramente (a menos que ele peça pra ignorar por enquanto).
5. Só recorra a `updated_time` de `/campaigns` (método mais fraco, só timing, sem autor nem direção) se `/activities` não retornar nada útil pro período.
6. Isso já foi usado com sucesso tanto pra explicar quedas de investimento de -50% a -75% que pareciam erro mas eram reestruturação em andamento, quanto pra descobrir um problema real de pagamento que passaria despercebido — sempre prefira essa checagem a perguntar pro Well "foi erro ou proposital?" quando a API já responde. Mas sempre com `actor_name` e checando `old_value`/`new_value` — sem isso, o risco é reportar uma ação que nunca aconteceu de verdade (ou que foi revertida no mesmo dia).

**Nunca assuma que a causa de um resultado bom continua sendo a mesma de check-ins anteriores sem reconferir.** Um caso real: check-ins anteriores da Clínica MWB atribuíam a melhora de custo/conversa às novas campanhas "Restrito" criadas numa reestruturação. Ao conferir `/act_<ID>/insights?level=campaign` com `actions` pra ver quantas conversas cada campanha trouxe essa semana, ficou claro que a campanha **antiga** (que nunca foi desligada, continuou rodando em paralelo) trazia a maioria das conversas (ex.: 74 de 95 do Márcio), não a campanha "Restrito" (só 12). A narrativa dos check-ins anteriores estava desatualizada/imprecisa. **Sempre que for atribuir um resultado bom (ou ruim) a uma campanha/ação específica, confirme com `insights` no nível de campanha (`level=campaign`, com `actions`) qual campanha realmente trouxe o resultado, em vez de repetir a explicação de uma semana anterior por inércia.**

**Cuidado com nomes de anúncio duplicados entre campanhas/conjuntos.** É comum o mesmo criativo (mesmo nome, ex.: "AD - VID - Dor nas costas") existir em várias campanhas/conjuntos diferentes (testes antigos, campanhas duplicadas, etc.), cada um com seu próprio `ad_id`. Bater só pelo nome pode pegar o ad_id errado (baixo gasto, resultado diferente do que está sendo descrito). Sempre que houver mais de um `ad_id` com o mesmo nome nos resultados de `insights`, confirme qual é o certo pelo **gasto/resultado da semana** (o que bate com o número que você está reportando) e, se disponível, pelo `campaign_id` de um evento relacionado no log de atividades — não pegue o primeiro que aparecer.

**Regra geral: todo anúncio citado pelo nome leva permalink.** Sempre que o rascunho mencionar um anúncio específico pelo nome/criativo em qualquer lugar do texto corrido (não só na lista dedicada de "melhores anúncios"), busque o permalink do Instagram dele (`/<ad_id>?fields=creative{instagram_permalink_url}`) e inclua junto, no mesmo formato de sempre (nome + link, dois-pontos, nunca travessão). Isso vale mesmo que o Well não tenha pedido a lista de "melhores anúncios" — se um anúncio é nomeado no texto, ele precisa ser rastreável até o criativo real. Exemplo: "o melhor resultado veio do vídeo 'X'" já exige achar o `ad_id` de "X" nos insights e anexar o link, mesmo fora de uma seção dedicada.

**Puxar os melhores anúncios da semana como lista dedicada, com link direto (permalink):**
Isso (a lista separada por etapa/pessoa) só entra quando o Well pedir explicitamente (ex.: "traz os melhores ads da semana com link"). Não é um passo padrão do fluxo de geração — não inclua a lista dedicada automaticamente num rascunho a menos que ele peça. Mas a regra geral acima (permalink pra qualquer anúncio citado no texto corrido) vale sempre, independente disso.
1. Insights no nível de anúncio pro período: `/act_<ID>/insights` com `level=ad`, `fields=ad_id,ad_name,campaign_name,spend,impressions,clicks,ctr,actions`, e o `time_range` do período do check-in.
2. **Identifique a etapa de funil de cada anúncio pelo nome da campanha** (procure "Topo de funil", "Meio de funil", "Fundo de funil" no `campaign_name`) e escolha o melhor anúncio **dentro de cada etapa**, usando a métrica que faz sentido pra aquela etapa, não uma métrica única pra tudo:
   - **Topo de funil** (tráfego/reconhecimento): melhor por `link_click` ou `video_view` (quem mais atraiu clique/visualização).
   - **Meio de funil** (engajamento/vídeo): melhor por `video_view` e engajamento (`post_reaction`, `post_engagement`).
   - **Fundo de funil** (conversa/lead): melhor por `onsite_conversion.total_messaging_connection` ou `onsite_conversion.messaging_conversation_started_7d` dentro de `actions` (conta de conversa) ou pela ação de lead relevante (conta de formulário).
   Cheque as três etapas quando existirem campanhas pra elas — não pare na primeira que achar resultado bom.
3. Pra cada anúncio escolhido, pegue o permalink do **Instagram** (não do Facebook) direto do criativo: `/<ad_id>?fields=creative{instagram_permalink_url}`. Isso já devolve a URL pronta tipo `https://www.instagram.com/p/<código>/` — não precisa montar nada manualmente.
4. **Formato de saída:** separe os anúncios por etapa de funil (Topo de Funil / Meio de Funil / Fundo de Funil — só mostre as etapas que tiverem anúncio na lista, não force as três) ou por pessoa (doutor/doutora), conforme o que o Well pedir. Dentro de cada grupo, liste **nome do anúncio + uma etiqueta curta entre parênteses dizendo o critério que fez ele ser escolhido + permalink do Instagram**, no formato `Nome do anúncio (critério): link` — **use dois-pontos, nunca travessão**, entre a etiqueta e o link (a proibição geral de travessão da skill vale aqui também). A etiqueta é qualitativa, não um número (ex.: "mais cliques da semana", "melhor custo por clique", "mais conversas da semana", "melhor CTR da semana") — **nunca coloque o valor da métrica** (nada de "CTR: 5,2%" ou "16 conversas"), só o rótulo do motivo. Isso ajuda o Well a saber por que aquele anúncio foi escolhido sem virar uma tabela de números.
- Sempre que usar dado extra da API num rascunho, deixe claro no rascunho (ou numa nota pro Well) que aquele ponto veio de consulta direta à API, não só do PDF — ajuda o Well a saber a origem se precisar conferir.

## Áudio do WhatsApp como contexto extra (transcrição local)

Se o Well anexar um áudio (`.opus`, `.m4a`, etc., normalmente um encaminhamento de WhatsApp explicando algo sobre a conta), transcreva localmente em vez de pedir pra ele digitar o conteúdo: `python3 -m whisper <arquivo> --language Portuguese --model small --output_format txt --output_dir <pasta do scratchpad>`. O pacote `whisper` já está instalado nesta máquina. Use a transcrição como contexto extra pro check-in/relatório (ela pode revelar informação que muda a narrativa, como já aconteceu com a Rock Encantech, onde o áudio revelou um bug de tracking que invalidava um resultado que parecia bom no PDF) — sempre trate o conteúdo do áudio como fonte de verdade a ser conferida contra os dados, não como algo a só resumir.

## Onde ficam os arquivos dos clientes

Esta skill é usada por vários gestores de tráfego da C2G, e cada um tem sua própria carteira de clientes numa pasta de Drive diferente — não existe um link fixo válido pra todo mundo, então a skill não guarda um link fixo aqui.

**Como descobrir o local, no início de uma conversa que precise de histórico de cliente:**
1. **Primeiro, confira se já existe `config.local.md`** na pasta desta skill (`~/.claude/skills/avaliador-checkin-relatorio-c2g/config.local.md`). Esse arquivo é local à máquina (está no `.gitignore`, nunca vai pro Git/GitHub) e guarda o link/caminho que o gestor já informou numa conversa anterior. Se existir, use o que estiver lá e não pergunte de novo.
2. **Se não existir ainda** (primeiro uso nesta máquina, ou o gestor nunca informou), **pergunte a ele**: o link do Drive (ou o caminho local, se o Drive estiver sincronizado nesta máquina) da pasta raiz onde ficam os check-ins semanais e relatórios mensais já existentes dos clientes dele, pra você usar como base/continuidade — já que cada gestor tem sua própria carteira e estrutura pode variar.
3. **Depois que ele informar, grave em `config.local.md`** (crie o arquivo se não existir) nesta mesma pasta da skill, com o link/caminho recebido, pra não precisar perguntar de novo nas próximas conversas nesta máquina. Grave só o link/caminho — nunca nome de cliente nem outro dado sensível nesse arquivo.

Dentro da pasta raiz que o gestor indicar, o padrão observado até agora (pode variar um pouco por gestor — confirme se a nomenclatura dele for diferente):
- `Checkin semanal/` — um Google Doc por cliente ativo, nomeado `[C2G] Checkin semanal - <Cliente>`. Inativos ficam em `Checkin semanal/Inativos/`.
- `Relatório mensal/` — mesma lógica, nomeado `[C2G] Relatório mensal - <Cliente>`, com `Relatório mensal/Inativos/` pros inativos.

Cada um desses docs **acumula o histórico inteiro** daquele cliente: a entrada mais recente fica no topo, separada das anteriores por uma linha `________`. Não existe um doc por semana/mês — é sempre o mesmo doc, com tudo dentro, em ordem cronológica decrescente (mais recente primeiro).

O nome do cliente no arquivo pode não bater 100% com o nome no cabeçalho de um PDF recebido (ex.: "Dra. Alexandra" no PDF vs. "Dra. Alexandra Cariello" no nome do arquivo) — combine pelo nome mais próximo; se houver ambiguidade real entre dois clientes parecidos, confirme com o gestor antes de prosseguir.

### Como ler o conteúdo desses docs

Se esta máquina tiver o Google Drive sincronizado localmente (verifique algo como `~/Library/CloudStorage/GoogleDrive-<email>/Meu Drive/<pasta configurada>/`), o caminho mais rápido é:
1. Achar o arquivo `.gdoc` do cliente na subpasta certa (nome bate com o padrão acima).
2. Extrair o `doc_id` de dentro dele: `grep -o '"doc_id":"[^"]*"' "arquivo.gdoc"`.
3. Buscar o texto puro sem precisar de login: `curl -sL "https://docs.google.com/document/d/<doc_id>/export?format=txt"`.

Se não houver sincronização local, use o navegador: abra o link salvo em `config.local.md`, entre na subpasta certa, abra o Doc do cliente e leia o conteúdo renderizado.

### Pasta "Dados de campanhas" — planilhas de dados brutos por conta

Dentro da pasta raiz existe também `Dados de campanhas/`, com planilhas (Google Sheets) de dados de campanhas isoladas, uma fonte a mais além da API e dos PDFs recebidos. Por enquanto só tem planilhas do **Google Ads** (Meta Ads deve vir depois). Use essa pasta quando precisar consultar algum dado específico de campanha que não veio no PDF nem é fácil de puxar da API do Meta (já que essas planilhas cobrem Google Ads, que não tem endpoint de API configurado nesta skill). Mesma técnica de leitura: se for `.gsheet` local, extrair o `doc_id` e buscar via `https://docs.google.com/spreadsheets/d/<doc_id>/export?format=csv`; sem sincronização local, abrir pelo navegador.

## Pasta "Entrada de relatórios" — inbox semanal de PDFs pra processar em lote

Dentro da pasta raiz (a mesma configurada em `config.local.md`, ver seção anterior) existe também, pra quem usa esse fluxo, uma subpasta `Entrada de relatórios/`. É o inbox onde o gestor sobe, toda segunda-feira, os PDFs brutos de todos os clientes da semana de uma vez — e apaga tudo no mesmo dia assim que os check-ins estiverem prontos.

Implicações práticas:
- **O conteúdo dessa pasta é efêmero.** Ela pode estar vazia, ter 1 arquivo ou ter 15+ arquivos, dependendo do momento da segunda-feira em que for consultada. Sumiu um arquivo que estava lá antes? Não é erro — o Well já processou e apagou. Não reclame de arquivo "faltando" nem tente usar como histórico.
- **Não confie só no nome do arquivo** para decidir semanal vs. mensal — os nomes vêm de um export automático e podem estar levemente errados (ex.: um arquivo chamado `checkin_semanal_...` mas com intervalo de datas de um mês inteiro, ou `relatorio_mensal_...` com intervalo de só 7 dias). O intervalo de datas de dentro do PDF (cabeçalho) é a fonte de verdade — sempre confirme por ali, mesmo que o nome do arquivo pareça claro.
- **Cada arquivo pode ser de um cliente diferente** — o nome do arquivo indica o cliente, mas combine com a lista de clientes em `Checkin semanal/` e `Relatório mensal/` pelo nome mais próximo (mesma regra de matching já descrita acima).

### Gerar todos os check-ins/relatórios de uma vez (processamento em lote)

Quando o Well pedir pra gerar/processar tudo da pasta de entrada de uma vez (ex.: "gera todos os checkins", "processa a entrada", "roda a segunda-feira"):

1. Liste todos os arquivos atualmente em `Entrada de relatórios/`.
2. Para cada arquivo, rode o Passo 1 de "Gerar check-in/relatório a partir de dados brutos" (identificar cliente, tipo e período pelo conteúdo do PDF, não só pelo nome do arquivo).
3. Para cada um, siga os Passos 2-4 normalmente (buscar histórico do cliente no Drive, aplicar a regra da 2ª semana quando se aplicar, escrever o rascunho) — **cada cliente é independente**, um erro ou ambiguidade num arquivo não deve travar o processamento dos outros.
4. Entregue os rascunhos de todos, **claramente separados e rotulados pelo nome do cliente** (ex.: um cabeçalho `### <Cliente>` antes de cada rascunho), na ordem em que os arquivos aparecem na pasta.
5. Continua valendo a regra do fluxo completo: essa etapa só entrega rascunhos — sem `Status` nem blocos. Blocos de cada cliente só saem depois que o Well revisar e reenviar o texto daquele cliente específico, e ele for aprovado.
6. Se algum arquivo tiver um problema real (cliente não identificável, período ambíguo, nome sem correspondência clara na lista de clientes), pule esse arquivo, sinalize o problema junto com os outros rascunhos, e continue com o resto — não pare o lote inteiro por causa de um arquivo problemático.

## Identificar o que foi recebido antes de agir

Sempre que o Well mandar um PDF/relatório (em vez de colar texto pronto), identifique primeiro:

**a) Check-in semanal ou relatório mensal?**
- Olhe o intervalo de datas do cabeçalho do relatório (algo como "dados analisados entre X e Y"). ~7 dias = check-in semanal. ~28-31 dias = relatório mensal.
- Cheque também o nome do arquivo e o título dentro do PDF, se disponíveis (costumam já indicar "Checkin semanal" ou "Relatório mensal").
- Se os sinais não baterem ou o intervalo for atípico, pergunte ao Well antes de seguir.

**b) É dado bruto ou já é texto corrido?**
- Um PDF de dashboard traz só números/tabelas (CPM, CTR, investimento, conversas, etc.), sem nenhuma frase explicando o "porquê". Isso é **dado bruto** — siga para "Gerar check-in/relatório a partir de dados brutos" abaixo.
- Se já tem parágrafos corridos respondendo as perguntas, é **texto já escrito** — segue direto o fluxo de avaliação (seções abaixo).

## Fluxo completo, do PDF aos blocos internos

Quando o ponto de partida é um PDF de dados brutos (não um texto já escrito), o processo tem etapas separadas — **nunca pule direto pra blocos**:

1. **Well manda o PDF de dados brutos.** A skill identifica o tipo (semanal/mensal), busca o histórico no Drive e gera só o **rascunho do texto** (ver "Gerar check-in/relatório a partir de dados brutos" abaixo). Nessa etapa: sem `Status`, sem blocos — só o rascunho pra revisão.
2. **Well revisa e reenvia o texto** (o rascunho ajustado, ou reescrito do zero por ele).
3. **A skill avalia esse texto reenviado** como no modo avaliação normal: confere as 6 perguntas e define `Status: Aprovado ✅` ou `Status: Reprovado ❌`.
4. **Só gera os 4 blocos internos se o status for Aprovado.** Se reprovado, a resposta pára nas correções — sem blocos — até o Well reenviar de novo um texto que passe na avaliação. Repita os passos 2-4 quantas vezes forem necessárias.

Esse fluxo em 4 passos vale tanto pra check-in semanal quanto relatório mensal. Se o Well já mandar o texto pronto direto (sem passar pelo PDF), o processo começa direto no passo 3.

## Gerar check-in/relatório a partir de dados brutos

Use este fluxo (passo 1 do fluxo completo acima) quando o Well mandar só o relatório de métricas (PDF/dashboard, dado bruto) e pedir pra gerar, montar ou rascunhar o check-in/relatório — não apenas avaliar um texto pronto.

**Passo 1 — Identificar cliente e período.** Tire do cabeçalho do PDF o nome do cliente e o intervalo de datas do período analisado.

**Passo 2 — Buscar o histórico do cliente no Drive** (ver seção acima):
- Abra o doc de **check-in semanal** desse cliente e leia pelo menos as últimas 2-3 entradas (mais recente no topo).
- Abra também o doc de **relatório mensal** desse cliente e leia pelo menos a entrada mais recente.
- Use esse material para dois fins:
  1. **Continuidade** — retome explicitamente pontos que ficaram em aberto na entrada anterior (pedidos ao cliente ainda não resolvidos, testes em andamento, pontos de atenção citados), em vez de tratar o período como isolado.
  2. **Padrão de comunicação daquele cliente** — cada cliente tem um jeito próprio (como o Well abre a mensagem, se usa @menção de alguém específico daquele cliente, nível de informalidade, nomenclatura recorrente de campanha). Replique esse padrão no rascunho ao invés de um tom genérico.

**Passo 3 — Regra da 2ª semana do mês.** Check-ins semanais só cobrem a 2ª, 3ª e 4ª semana do mês — a 1ª semana é coberta pelo relatório mensal, não por um check-in semanal separado. Então, se ao abrir o doc de check-in semanal a entrada mais recente for da **última semana do mês anterior** (ou seja, falta uma entrada cobrindo a 1ª semana do mês atual), isso é esperado — **não é doc desatualizado nem erro**. Nesse caso, combine como contexto:
- a última entrada do check-in semanal (última semana do mês anterior), **e**
- a entrada mais recente do relatório mensal (o mês anterior fechado).

Ambas juntas dão o contexto completo antes de escrever o check-in da 2ª semana.

**Passo 4 — Escrever o rascunho.** Com os dados novos do PDF + o contexto dos Passos 2-3:
- **Sempre comece pela visão geral da conta, sem exceção** — não só pelas campanhas/especialidades individuais. Se o PDF trouxer um bloco consolidado (ex.: "GERAL DA CONTA", totais de Google Ads e/ou Meta Ads antes do detalhamento por campanha/especialidade), reflita esse consolidado no início do check-in antes de entrar no detalhe por campanha — o Well sempre dá o panorama geral primeiro, o detalhe por campanha vem depois pra ilustrar o que puxou esse resultado. Isso vale tanto pro rascunho gerado quanto como critério ao avaliar um texto que o Well mandou: se o texto pular direto pras campanhas sem dar o panorama geral primeiro, aponte como correção.
- **Só cite valor investido/verba gasta quando for muito relevante** (uma mudança grande de investimento, tipo ±50% ou mais) — não repita o número de "valor investido" toda vez só porque ele está no PDF. Isso é diferente de **custo por resultado** (custo por conversão, por conversa, por lead), que é uma métrica de eficiência e **sempre** deve vir com número, como já estabelecido. A distinção é: quanto foi gasto raramente importa pro cliente por si só; quanto custou cada resultado, sim. Exceção: se a variação de investimento for a própria causa de outra métrica mudar (ex.: conta pausada por dias, corte de verba explicando queda de volume), aí o número de investido entra como parte da explicação do porquê, não como dado solto.
- Responda as 6 perguntas (adaptando Q4/Q5 se for relatório mensal — ver seção abaixo) no tom/padrão daquele cliente específico.
- **Dê continuidade real aos pontos em aberto** identificados no Passo 2 — não é só mencionar que existe uma pendência, é continuar exatamente o mesmo assunto/pergunta que ficou em aberto. Releia a pendência anterior com atenção antes de formular a nova versão dela: se a pergunta anterior foi sobre X, a pergunta de continuidade também precisa ser sobre X, não sobre um tópico parecido ou adjacente que a skill achou relevante essa semana. Não invente uma pendência nova que pareça combinar com o dado da semana se ela não tiver relação direta com o que ficou pendente antes.
- **Nunca escreva "vou investigar" (ou equivalente) como resposta a uma queda ou anomalia.** Isso é preguiça de análise — o rascunho precisa propor uma **causa provável**, com base nos próprios números do PDF, e a partir dela um próximo passo concreto. Exemplo: se conversões caíram mas CPM e CTR ficaram estáveis, a causa provável é queda na conversão pós-clique (público ou página), não leilão — diga isso, não "vou investigar o que houve". Se genuinamente não há dado suficiente no PDF pra formular uma hipótese, aí sim diga que vai olhar mais de perto, mas isso deve ser exceção, não o padrão.
- **Antes de montar o P2 (ação já executada) ou de perguntar pro Well o que ele fez, sempre confira primeiro o log de atividades da conta** (`/act_<ID>/activities`, ver seção "Acesso à API do Meta Ads" acima) pra ver se teve alguma ação manual real na semana. Não é opcional nem só pra quando a queda for grande — é o passo padrão antes de: (a) deixar o P2 em branco, (b) perguntar pro Well "o que você fez essa semana?", ou (c) assumir que não houve ação nenhuma. Só pergunte pro Well depois de já ter checado e não ter achado nada de relevante (lembrando de filtrar automações/regras, que não contam como P2 — ver regras do `event_type` na seção da API).
- **Todo resultado ruim precisa vir com o porquê, nunca só o fato.** Não basta dizer "contatos caíram, custo subiu" — precisa dizer *o que explica* essa queda, usando as outras métricas do próprio PDF (CPM, CTR, alcance, frequência, cliques) pra montar a explicação. Ex.: contatos caindo + CPM subindo + alcance caindo = "tivemos menos gente vendo o anúncio, e ainda pagando mais caro pra isso". Se as métricas do PDF não bastarem pra montar uma explicação real (ex.: o PDF não detalha por campanha o suficiente), **consulte a Graph API do Meta Ads diretamente** (ver seção "Acesso à API do Meta Ads" acima) pra olhar as campanhas daquela conta no período analisado antes de escrever — não deixe a queda sem explicação nem generalize algo que os dados não sustentam.
- **Respeite o tamanho médio real** (ver "Tamanho esperado do texto" abaixo) — não escreva um textão nem um rascunho raso demais.
- **Nunca use travessão longo e evite vícios de IA** (ver "Revisão de vícios de IA e travessão" abaixo) — o rascunho já precisa nascer limpo disso, sem depender de uma correção depois.
- Deixe claro que é um **rascunho para revisão do Well**, não texto final pronto pra envio.
- **Pare aqui.** Não gere `Status: Aprovado/Reprovado` nem os 4 blocos internos nessa etapa — isso só acontece depois que o Well revisar/ajustar e reenviar o texto (ver "Fluxo completo" acima). Entregue só o rascunho.

### Tamanho esperado do texto (baseado no histórico real)

Levantamento feito sobre **443 entradas reais** de check-in/relatório de todos os clientes (ativos e inativos), contando caracteres de cada entrada individual nos docs do Drive:

- **Check-in semanal** (347 entradas analisadas): média **~1.620 caracteres**, mediana 1.563. Faixa típica (a maioria dos check-ins do Well cai aqui): **~1.300 a ~1.900 caracteres**.
- **Relatório mensal** (96 entradas analisadas): média **~1.770 caracteres**, mediana 1.762. Faixa típica: **~1.400 a ~2.000 caracteres**.

Use essas faixas como alvo ao escrever o rascunho: nem um textão de dado bruto (o que já é proibido por outra regra desta skill), nem um resumo raso demais que não dá conta das 6 perguntas com contexto. Casos legítimos podem sair fora da faixa (uma semana com muita coisa acontecendo, ou uma semana tranquila) — não corte contexto relevante só pra bater o número, é uma referência de calibragem, não um limite rígido.

## Por que essas 6 perguntas existem

O objetivo do check-in não é entregar **dado bruto** (número solto, sem contexto — ex.: "CPM: 12,40"), e sim **informação** (dado + contexto que gera entendimento — ex.: "CPM caiu 15%, ou seja, o leilão ficou mais barato"). Os três objetivos por trás de todo check-in/relatório são: (1) informar o cliente sobre o que foi feito e o estado atual da conta, (2) prestar satisfação do uso do investimento do cliente, (3) trazer tranquilidade e confiança, mostrando que existe uma linha de raciocínio (uma "tese") por trás das decisões, não ações aleatórias semana a semana. Isso não é motivo formal de reprovação por si só, mas orienta as sugestões de melhoria.

## As 6 perguntas obrigatórias

A ordem é livre. Cobertura mínima é suficiente — não precisa ser extenso.

**Check-in semanal:**
1. Qual o problema da conta que vocês estão tentando resolver?
2. O que você fez pra resolver esse problema?
3. Qual o resultado obtido do teste que você fez? (um por um, se fizer sentido, ou de forma macro)
4. Qual o resultado da **semana**, comparado com a semana anterior? E por quê?
5. O que você vai fazer **essa semana** pra tentar melhorar/resolver o resultado?
6. O que você precisa do cliente?

**Relatório mensal** — mesma lógica, adaptando apenas Q4 e Q5:
4. Qual o resultado do **mês**, comparado com o mês anterior? E por quê?
5. O que você vai fazer **no próximo mês** pra tentar melhorar/resolver o resultado?

## Regras de avaliação

- **P6 não é obrigatório.** Se não houver nada a pedir ao cliente naquela semana/mês, isso não é motivo de reprovação — não force um pedido artificial.
- **P2 precisa ser ação já executada (passado).** "Vou trocar a campanha" não satisfaz P2 — isso é P5 (plano futuro). P2 exige algo que já foi feito.
- As causas mais comuns de reprovação são justamente P2 (nenhuma ação explícita no passado) e P4 sem comparação/motivo.
- Dado bruto sem contexto (CTR, CPM, impressões soltos, sem explicar o que significam) não é, por si só, motivo de reprovação formal — mas é um ponto de melhoria a apontar, porque não passa "informação" de verdade ao cliente (ver seção acima).
- Nunca tratar aumento de custo como boa notícia — se o texto fizer isso, aponte como correção.
- Proibido no processo da C2G (ver SOP): enviar métricas que não foram usadas na argumentação, e mandar relatório gerado por ferramenta automática (tipo ChatGPT) sem revisão/voz humana — ver "Revisão de vícios de IA e travessão" abaixo para o checklist completo de como identificar isso. Se o texto tiver vários sinais de "cara de IA" ao mesmo tempo, trate como motivo de reprovação; 1-2 ocorrências isoladas viram sugestão de estilo.
- O texto precisa abrir com a visão geral da conta antes de entrar em campanha/especialidade específica (ver regra no Passo 4 de geração) — se não abrir assim, aponte como correção. E valor investido/verba só deve aparecer quando for uma mudança grande (±50% ou mais) ou quando for a própria explicação de outra métrica — fora isso, é sugestão de estilo cortar, não motivo de reprovação.

## Formato da avaliação

Sempre abrir a resposta com uma destas linhas, exatamente:

```
Status: Aprovado ✅
```
ou
```
Status: Reprovado ❌
```

Se **aprovado**: não é preciso justificar cada pergunta uma por uma — só confirmar e seguir direto para os blocos.

Se **reprovado**: apontar objetivamente qual(is) pergunta(s) faltou/faltaram (ex.: "Faltou P2 — você diz o que vai fazer, mas não o que já foi feito") e como resolver, em 1-3 linhas por ponto. Sem textão, sem repetir de volta o check-in inteiro do Well. **Pare por aí — não gere os blocos.** Eles só saem depois que o Well reenviar o texto ajustado e ele passar como Aprovado.

## Os 4 blocos internos

**Regra principal: só monte os blocos se o status for `Aprovado ✅`.** Se reprovado, não gere os blocos nessa rodada — espere o Well reenviar o texto corrigido, reavalie do zero, e só então (se aprovado) monte os blocos. Isso vale tanto pra texto avaliado direto quanto pra um rascunho gerado a partir de PDF (ver "Fluxo completo" acima).

Exceção: se o Well pedir explicitamente pra montar os blocos mesmo com o texto reprovado (ex.: "monta os blocos mesmo assim"), atenda o pedido — mas deixe claro na resposta que o check-in segue reprovado e que os blocos estão sendo montados por pedido explícito, fora do fluxo padrão.

Uma vez aprovado, monte sempre os 4 blocos abaixo, para o Well colar no formulário interno da agência:

- **Bloco 1:** Qual foi o principal problema/oportunidade identificada [na semana passada / no mês passado] e o que foi feito para resolver
- **Bloco 2:** Quais foram os resultados das ações realizadas e qual é o estado atual da conta
- **Bloco 3:** O que será feito a partir de agora para tirar a conta do estado atual
- **Bloco 4:** O que precisamos da parte do cliente neste momento e por quê

Regras dos blocos:
- Os blocos refletem **o texto que o Well enviou**, não uma versão corrigida — mesmo se reprovado, monte com o que foi mandado, a menos que ele peça explicitamente para usar a versão sugerida/corrigida.
- Se o Well disser "monte com o que eu mandei", isso confirma a regra acima: use o texto original tal como está, sem adicionar nem corrigir nada.
- Se P6 estiver ausente (sem pedido ao cliente), o Bloco 4 deve dizer algo como "Sem pendências da parte do cliente no momento" — não invente um pedido.
- **Relatório mensal:** adicione a tag `[RELATÓRIO MENSAL]` no início do texto do Bloco 1, para o sistema da agência identificar o tipo de entrada. Check-ins semanais não levam tag.
- Se o Well pedir só os blocos (sem pedir avaliação explicitamente), **avalie mesmo assim antes de montar** — mas mostre só o resultado da avaliação de forma resumida (ex.: "Aprovado ✅" numa linha) seguido dos blocos, sem o detalhamento didático de correções. Se reprovar, siga a regra principal acima: aponte as correções e não monte os blocos.

## Tom das cobranças ao cliente (P6) — sempre extremamente suave

Quando o pedido ao cliente (P6) é uma pendência repetida (já foi pedida em check-ins anteriores), o tom **nunca** pode soar como cobrança ou impaciência. Isso vale tanto pra rascunhos gerados quanto pra sugestões numa avaliação.

- **Nunca** conte ou mencione quantas vezes já foi pedido (nada de "já é a terceira semana que peço isso", "segue pendente há semanas", "ainda estou esperando faz tempo"). Isso soa como cobrança e não é o tom do Well com os clientes.
- Prefira retomar o pedido de forma leve, como se fosse a primeira vez perguntando, só dando o contexto de continuidade sem cobrança: "seguimos precisando de X" / "ainda fico no aguardo de X" / "assim que puder, consegue trazer X?" — direto e educado, sem peso.
- Nunca use linguagem de urgência forçada ou passivo-agressiva ("preciso urgentemente", "isso está travando o trabalho", "sem isso não consigo avançar") a menos que seja genuinamente crítico e o próprio Well tenha sinalizado isso nos dados.
- Ao avaliar um texto que o Well mandou com esse tom mais cobrador, aponte como sugestão de tom (não é motivo de reprovação formal, é um ajuste de estilo).

## Tom e boas práticas do processo C2G (contexto para as sugestões, não critério formal de reprovação)

- PT-BR informal e direto, sem jargão sem explicação — é assim que o Well se comunica com os clientes.
- Métricas sempre com contexto e variação percentual, nunca soltas.
- Comemorar vitórias logo de cara quando houver boa notícia.
- Evitar textão de dado bruto — dashboards existem pra isso; o check-in é pra dar inteligência, não dado.
- Se o Well perguntar algo como "tem certeza que tá certo?" sobre um número, refaça o cálculo a partir dos dados brutos que ele forneceu antes de responder.

**Vocabulário extremamente simples e popular, nunca de relatório corporativo.** Isso é diferente de vício de IA (seção abaixo): é nível de formalidade. Mesmo uma frase gramaticalmente correta e sem clichê de IA pode soar errada por ser "bonita" ou formal demais para como o Well realmente fala. Sempre trocar a palavra mais rebuscada pela mais simples e do dia a dia. Exemplos de troca (o padrão é o da direita):
- "resultado sólido" / "seguiu sólido" → "continuou bom" / "foi bem"
- "campanha X seguiu evoluindo" → "campanha X teve uma semana positiva" / "campanha X melhorou"
- "apresentou um desempenho satisfatório" → "foi bem" / "teve um resultado bom"
- "performance robusta" → "resultado forte"
- "cenário desafiador" → "semana mais difícil" / "semana mais complicada"
- "consolidado da conta" (como substantivo pomposo) → "no total" / "somando tudo"
- "expressivo crescimento" → "cresceu bastante" / "cresceu muito"

**Teste rápido:** leia a frase como se fosse gravar um áudio de WhatsApp pra um amigo. Se é uma palavra que ninguém usaria numa conversa normal falada, troque por uma mais simples, mesmo que ela pareça "mais profissional" ou "mais bem escrita". Aplicar isso tanto ao escrever um rascunho quanto ao dar sugestões de correção pra um texto do Well.

## Revisão de vícios de IA e travessão (baseado na skill copywriter-supremo)

Essa checagem roda **sempre**, em todo texto de check-in/relatório que passar pela skill — tanto num texto que o Well mandou pra avaliar quanto num rascunho que a própria skill está escrevendo (ver Passo 4 de geração acima). Fonte completa: `references/vicios-de-ia-checklist.md`.

**Nunca use travessão longo (—).** Em nenhum lugar do texto. Troque por ponto, vírgula, dois-pontos ou parênteses.

**Evite ativamente os sinais clássicos de texto gerado por IA sem revisão humana:**
- Conectores previsíveis abrindo frase/parágrafo ("além disso", "portanto", "ou seja", "no entanto", "vale ressaltar", "dito isso", "em suma", "dessa forma").
- Aberturas e fechamentos de fórmula genérica ("a verdade é que", "chega de", "em resumo", "concluindo") — em vez disso, siga o padrão real do Well: abre com "Olá, pessoal! Tudo bem? Segue o check-in..." e fecha com algo direto tipo "Qualquer dúvida, só chamar!".
- Estruturas de contraste viciadas ("não é apenas X, é Y", "e o melhor de tudo?").
- Adjetivos inflados sem número junto ("resultado incrível", "crescimento surpreendente") — sempre trocar por dado concreto ("conversas subiram 40%").
- Listite: transformar o check-in em bullet points quando o padrão real do Well é parágrafo corrido.
- Reticências decorativas e emoji em excesso no corpo do texto.

**Ao gerar um rascunho (Passo 4):** a skill precisa escrever já respeitando essas regras — não faz sentido produzir um rascunho cheio de vício de IA pra depois reprovar o próprio texto. Antes de entregar o rascunho, releia e corrija qualquer ocorrência.

**Ao avaliar um texto do Well:** se aparecer 1-2 ocorrências isoladas, aponte como sugestão de estilo (não reprova sozinho). Se o texto tiver **vários** desses sinais ao mesmo tempo (cara de "colado direto de um ChatGPT sem revisão"), isso conta como violação da regra do SOP que proíbe mandar relatório gerado por IA sem voz humana (ver "Regras de avaliação" abaixo) — trate como motivo de reprovação, junto com as 6 perguntas.

## Estrutura da resposta

**Se o ponto de partida é um PDF de dados brutos (passo 1 do fluxo completo):**
1. Só o rascunho do check-in/relatório, sinalizado como rascunho para revisão.
2. Nada de `Status` nem blocos nessa etapa — eles vêm depois, quando o Well reenviar o texto revisado.

**Se avaliando um texto (já escrito, ou reenviado depois do rascunho):**
1. `Status: Aprovado ✅` ou `Status: Reprovado ❌`
2. Se reprovado: as correções objetivas, e **parar por aí — sem blocos**.
3. Se aprovado: os 4 blocos internos, rotulados claramente (Bloco 1, Bloco 2, Bloco 3, Bloco 4).

Não adicione seções extras, resumos ou repetição do texto original além do necessário para apontar correções.

## Exemplo rápido

**Entrada (reprovado):**
> "Essa semana o CPM caiu bastante e o CTR subiu. Vamos continuar otimizando os criativos."

**Saída esperada:**
```
Status: Reprovado ❌

Faltou:
- P1/P2: não diz qual problema estava sendo resolvido nem o que foi feito (só descreve métricas soltas).
- P4: falta comparar com a semana anterior e explicar o porquê da queda de CPM/alta de CTR.
- P6: não diz o que precisa do cliente (ok se realmente não houver pendência).

Como ajustar: recapitule o problema da semana passada, diga a ação tomada (não só "otimizando"), e compare o resultado com a semana anterior com números — ex.: "CPM caiu de X para Y (-15%), então o leilão ficou mais barato e por isso o CTR subiu."
```
(seguido dos 4 blocos, montados com o texto original do Well)
