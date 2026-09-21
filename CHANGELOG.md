# Changelog

Todas as mudanças relevantes na skill ficam registradas aqui, mais recente primeiro.

## 2026-09-21

- Planilha do Google Ads por campanha: a skill agora explica que é opcional e pede ao gestor se ele tem uma, como ler as várias abas (`htmlview` pra achar os `gid`), como achar a aba do cliente pelos nomes das campanhas do PDF e conferir os totais, o que ela serve, e os limites (defasagem de 1 a 2 dias e nenhum histórico de alterações, então nunca afirmar ação do gestor no Google a partir dela).

- Sem token do Meta Ads: a skill agora avisa logo, antes do primeiro rascunho, que o token é muito recomendado, explica o que se perde sem ele (log de alterações, causa por campanha, alerta de pagamento, links dos anúncios, estado atual) e como seguir só com o PDF sem inventar ação nem causa.

- Regra de tamanho: a contagem de caracteres agora vale pro texto inteiro que vai pro cliente, incluindo a lista de melhores anúncios com links. Um rascunho tinha sido medido só pelo corpo (1.715, dentro da faixa) e saiu com 2.819 no total, o que estourava o limite. Passou a valer: medir o texto completo antes de entregar e cortar se passar de ~1.900 (gancho do anúncio em vez do nome completo, 1 anúncio por etapa de funil).

## 2026-09-14 (4)

- Nova seção "Como o gestor pode te mandar os dados de um cliente", listando todos os formatos aceitos: PDF anexado direto no chat, lote via pasta "Entrada de relatórios" do Drive (qualquer gestor pode criar a sua), print/imagem com dados, texto já escrito, ou áudio do WhatsApp como contexto extra.
- Padronizada a linguagem de "Well" pra "gestor" nas seções de identificação de PDF e fluxo completo, já que a skill agora é usada por vários gestores com carteiras diferentes.

## 2026-09-14 (3)

- Adicionado passo a passo pra gerar um token do Meta Ads do zero (Usuário do Sistema no Business Manager, com token que não expira em 60 dias) pra gestores que ainda não têm um configurado.
- Adicionada instrução de onde salvar o token gerado (`Token do Meta Ads.pdf` na raiz da pasta configurada em `config.local.md`) pra skill conseguir achar e usar.

## 2026-09-14 (2)

- Removido o link fixo da pasta do Drive de dentro de `SKILL.md` — cada gestor de tráfego tem sua própria carteira de clientes e sua própria pasta, então um link fixo não fazia sentido num repositório compartilhado publicamente.
- A skill agora pergunta ao gestor, na primeira vez que precisar do histórico de um cliente, onde ficam os check-ins semanais e relatórios mensais já existentes dele, e salva a resposta em `config.local.md` (arquivo local, no `.gitignore`, nunca vai pro GitHub) pra não perguntar de novo nas próximas conversas na mesma máquina.

## 2026-09-14

Primeira versão publicada, consolidando o aprendizado de um dia inteiro de uso real processando o lote semanal de check-ins/relatórios.

- Geração de rascunho a partir de PDF de dados brutos, com detecção automática de semanal vs. mensal pelo intervalo de datas (não pelo nome do arquivo).
- Uso do histórico do cliente no Drive (docs acumulativos por cliente) como contexto de continuidade e tom.
- Regra da 2ª semana do mês: combinar o último check-in semanal com o último relatório mensal quando a semana 1 já foi coberta pelo mensal.
- Gate de aprovação: os 4 blocos internos só são montados se o status for Aprovado.
- Acesso à API do Meta Ads (Graph API) pra dados que o PDF não cobre — com regra de segurança pro token (nunca persistido em arquivo, nunca exibido).
- Metodologia de diagnóstico via log de atividades da conta (`/activities`): sempre checar `actor_name` (filtrar automações da Meta), sempre reconstruir `old_value`/`new_value` cronologicamente até o estado final antes de reportar uma ação como P2.
- Regra: nunca repetir a atribuição de um resultado bom/ruim de check-ins anteriores sem reconferir via `insights?level=campaign` no período atual.
- Regra: qualquer anúncio citado pelo nome no texto leva permalink do Instagram; formato padronizado pra lista dedicada de melhores anúncios (por etapa de funil ou por pessoa, com etiqueta qualitativa, dois-pontos em vez de travessão).
- Regra: cuidado com nomes de anúncio duplicados entre campanhas — confirmar pelo gasto/resultado do período, não pegar o primeiro `ad_id` que aparecer.
- Transcrição local de áudios de WhatsApp anexados (Whisper) como contexto extra — usado pra revelar um bug de tracking que invalidava um resultado aparentemente bom.
- Pasta "Dados de campanhas" no Drive como fonte extra de dados brutos (planilhas de Google Ads).
- Tamanho esperado do texto calibrado por estudo de 443 entradas históricas reais (semanal ~1300–1900 caracteres, mensal ~1400–2000).
- Checklist de vícios de escrita de IA e proibição de travessão, adaptado da skill `copywriter-supremo`.
- Tom sempre suave nas cobranças ao cliente (P6), sem contar quantas vezes já foi pedido.
- `references/contas-meta-ads.md`: em vez de manter uma lista fixa cliente → conta (removida por privacidade, já que este repositório é público), a skill agora sempre consulta `/me/adaccounts` sob demanda e documenta como escolher entre contas parecidas/duplicadas.
