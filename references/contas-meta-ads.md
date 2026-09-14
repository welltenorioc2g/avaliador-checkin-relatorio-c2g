# Como achar a conta de anúncios Meta Ads de um cliente

Esta skill não mantém uma lista fixa de cliente → ID de conta (evita vazar nomes de clientes em texto plano num repositório compartilhado, e a lista desatualiza toda hora — conta nova, conta desativada, cliente que saiu).

Sempre que precisar do `act_<ID>` de um cliente:
1. Consulte `/me/adaccounts?fields=id,name,account_status` com o token do momento (ver seção "Acesso à API do Meta Ads" no SKILL.md para o protocolo de uso do token).
2. Ache pelo nome mais próximo em `name` — o nome cadastrado na Meta pode não bater 100% com o nome do cliente no Drive (variações comuns, ex.: sigla, "CA 01/02/03" de contas duplicadas).
3. Se houver mais de uma conta pro mesmo cliente (comum — contas antigas, testes, contas por especialidade), confira `account_status` (1 = ativa, 2 = desativada, 3 = não configurada/pendente, 9 = em período de carência, 101 = pendente de encerramento) e o histórico recente da conta (`/act_<ID>/activities`) pra confirmar qual é a que está rodando de verdade antes de puxar dados dela.
4. Se ainda houver ambiguidade real entre duas contas parecidas, pergunte ao Well antes de prosseguir.
