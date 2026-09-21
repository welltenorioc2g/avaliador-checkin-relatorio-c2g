# avaliador-checkin-relatorio-c2g

Skill do [Claude Code](https://claude.com/claude-code) da agência C2G (2Gather) pra gerar e avaliar check-ins semanais e relatórios mensais de tráfego pago (Meta Ads/Google Ads) enviados a clientes, e montar os blocos internos usados no formulário da agência.

Cobre, num fluxo só:
- Gerar rascunho a partir de um PDF de dados brutos, usando o histórico do cliente no Drive e dados ao vivo da API do Meta Ads como contexto.
- Avaliar um check-in/relatório já escrito contra as 6 perguntas obrigatórias do processo da C2G, com Status Aprovado/Reprovado.
- Montar os 4 blocos internos (problema/ação, resultado/estado atual, próximos passos, pedido ao cliente) só depois de aprovado.
- Checar vícios de escrita de IA e proibição de travessão.
- Processar em lote a pasta "Entrada de relatórios" do Drive.

Todo o funcionamento está documentado em [`SKILL.md`](SKILL.md); o racional completo e exemplos comentados estão em [`references/`](references/).

## Como instalar

Clone esta pasta pra dentro de `~/.claude/skills/` na sua máquina (clonar, e não só baixar o zip, é o que permite receber atualizações depois):

```bash
git clone https://github.com/welltenorioc2g/avaliador-checkin-relatorio-c2g.git ~/.claude/skills/avaliador-checkin-relatorio-c2g
```

O Claude Code reconhece a skill automaticamente a partir daí (invocação automática por contexto, ou manual com `/avaliador-checkin-relatorio-c2g`).

## Configuração por gestor

Cada gestor de tráfego tem sua própria carteira de clientes e sua própria pasta no Drive — por isso este repositório não guarda nenhum link de Drive. Na primeira vez que você usar a skill, ela vai perguntar o link (ou caminho local) da pasta com os check-ins/relatórios já existentes dos seus clientes, e salvar a resposta em `config.local.md`, um arquivo local (no `.gitignore`, nunca sobe pro GitHub) que evita repetir a pergunta nas próximas conversas.

## Duplicar/adaptar pra outro contexto

Esta skill assume as convenções específicas da C2G (estrutura de pastas no Drive, as 6 perguntas, os 4 blocos internos, tom de voz). Pra adaptar pra outra agência ou processo, dá pra usar como ponto de partida — mas troque as referências de pasta, o vocabulário e os critérios de avaliação em `SKILL.md` e `references/`.

## Como receber atualizações

Só o Well atualiza este repositório por enquanto. Mudanças são documentadas em [`CHANGELOG.md`](CHANGELOG.md) e no histórico de commits.

Quem clonou recebe as mudanças com um comando dentro da pasta da skill:

```bash
cd ~/.claude/skills/avaliador-checkin-relatorio-c2g && git pull
```

- **A skill avisa quando há versão nova.** Na primeira vez que ela é usada numa conversa, confere o GitHub e, se houver atualização, te avisa e pergunta se quer atualizar (só atualiza se você disser que sim). Se preferir, rode o comando acima quando quiser, ou peça pro Claude Code "atualiza a skill".
- **Seu `config.local.md` não é afetado.** Ele está no `.gitignore`, então seu link do Drive continua no lugar depois de atualizar.
- **Não edite os arquivos da cópia clonada** (`SKILL.md`, `references/`). Se você mexer neles, o `git pull` pode dar conflito. Se tiver uma sugestão, avise o Well ou faça um fork.
- **Quem baixou só o arquivo `.skill` (zip) não recebe atualização**: precisa baixar de novo. Por isso o jeito recomendado é clonar.
- Pra ver o que mudou desde a última vez, leia o [`CHANGELOG.md`](CHANGELOG.md).
