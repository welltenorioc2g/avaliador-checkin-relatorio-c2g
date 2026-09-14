# Changelog

Todas as mudanças relevantes na skill ficam registradas aqui, mais recente primeiro.

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
