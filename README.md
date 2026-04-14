# Make Central Brain

Base do ecossistema multiagente da Make.

Este projeto foi ajustado para o modelo correto de operacao:

- o operador fala com o Codex em texto comum
- o sistema faz a proxima pergunta sozinho
- o onboarding e preenchido por conversa
- os agentes certos sao ativados automaticamente
- aprovacoes tambem acontecem por texto comum

Comandos e scripts continuam existindo, mas sao ferramentas internas de manutencao e validacao. Eles nao sao a experiencia esperada para a operacao diaria.

## O Que o Sistema Faz

O Make Central Brain conecta 3 camadas:

1. `Memoria oficial em Markdown`
   Tudo que o sistema sabe sobre um cliente fica em `make-central-brain-context/clients/<slug>/`.

2. `Control plane`
   O core em Python guarda fila, jobs, sessoes de conversa, aprovacoes, artefatos, eventos e indexacao do contexto.

3. `Agentes e skills`
   Cada pilar tem agentes proprios. O onboarding ativa so o que faz sentido naquele momento.

## Como Deve Ser Usado na Pratica

### Regra principal

O operador nao deve precisar decorar comandos.

O fluxo desejado e este:

1. A pessoa escreve normalmente para o Codex.
2. O Codex faz a pergunta seguinte.
3. O Codex salva o que foi respondido.
4. Quando ja houver contexto suficiente, o Codex ativa o cliente automaticamente.
5. Se MakeADS estiver contratado, o Codex puxa o kickoff de growth.
6. O plano de midia aprovado vira a diretriz das campanhas.
7. Os proximos agentes entram em acao com base nisso.

## Fluxo de Ponta a Ponta

### 1. Conversa inicial

O operador pode escrever algo como:

```text
Novo cliente.
Nome: Clinica Sorriso Prime
Servicos: makeads, social, crm, mavi
Meta: gerar 120 leads qualificados
```

Se faltarem dados, o sistema pergunta o proximo item sozinho.

Regras dessa etapa:

- assim que nome, slug e servicos contratados estiverem claros, a transcricao da R1 vira prioridade
- o sistema deve pedir a R1 antes de empurrar o operador para etapas que dependem de estrategia

Exemplos de perguntas automaticas:

- qual e a transcricao da R1?
- qual e a verba mensal de midia?
- quais ofertas entram primeiro?
- o cliente usa MakeCRM?
- a Meta ja esta conectada?

### 2. Coleta de contexto

O sistema vai preenchendo:

- nome do cliente
- slug
- servicos contratados
- meta de 90 dias
- metricas de sucesso
- investimento
- verba de midia
- ofertas
- ticket
- ciclo de vendas
- capacidade comercial
- transcricao da R1
- concorrentes
- regioes
- tom de voz
- ativos
- acessos
- aprovadores

Isso e feito por conversa, nao por terminal.

### 3. Bootstrap automatico

Quando o contexto minimo estiver completo, o sistema:

- cria a pasta do cliente
- gera os arquivos canônicos
- cria os arquivos de canal em `04_channels/`
- grava a R1 no historico
- indexa o contexto
- abre os primeiros jobs

Tudo isso ja esta implementado no bootstrap do onboarding.

### 4. Regra obrigatoria de growth

Se `makeads` estiver contratado:

- o primeiro job obrigatorio sera `onboarding-media-plan`
- esse job usa a skill oficial `plano-de-midia-make`, versionada dentro do proprio `Make Brain`
- o plano de midia aprovado vira a diretriz mestra do growth
- so depois entram `campaign-roadmap`, criativo, performance e expansao

Em resumo:

`R1 -> plano de midia -> diretriz de growth -> campanhas`

### 5. Aprovacao do plano

Depois que o growth gerar o plano de midia, o operador nao precisa rodar comando tecnico.

Ele pode simplesmente escrever:

```text
Aprovado. Pode seguir.
```

E depois colar:

1. o conteudo final do plano
2. um resumo curto da diretriz das campanhas

O ciclo correto agora e:

1. o sistema aguarda o plano gerado com `plano-de-midia-make`
2. quando o plano chega, ele pergunta se esta aprovado ou quais mudancas sao necessarias
3. se houver mudancas, ele volta para revisao
4. so depois da aprovacao ele pede o resumo executivo
5. entao publica o plano e segue para a proxima etapa

Quando isso acontece, o sistema:

- publica o plano em `08_artifacts/`
- atualiza `04_channels/makeads.md`
- fecha o job `onboarding-media-plan`
- registra o evento no control plane

### 6. Execucao do restante

Depois disso:

- o contexto do cliente fica pronto
- o canal MakeADS ja fica orientado pelo plano aprovado
- os proximos jobs do growth podem seguir
- os outros pilares usam a mesma memoria oficial

## Skill Oficial de Growth

O projeto agora carrega a skill oficial de plano de midia dentro do proprio repositorio em:

`make-central-brain-core/brain/skills/makeads/plano-de-midia-make/SKILL.md`

Fallback legado, se necessario:

`C:\Users\Nicolas\.codex\skills\plano-de-midia-make\SKILL.md`

Ela faz parte do processo de growth.

Ela nao e opcional quando `makeads` estiver contratado.

Funcao dela no ecossistema:

- ler a R1
- gerar o plano de midia
- dar a diretriz inicial das campanhas
- servir de base para o roadmap de growth

## Estrutura de Contexto

Cada cliente fica em:

```text
make-central-brain-context/
  clients/
    <slug>/
      00_profile.md
      01_brand_voice.md
      02_business_model.md
      03_offer_stack.md
      04_channels/
      05_sales_crm.md
      06_mavi.md
      07_history/daily/
      08_artifacts/
      09_rules_and_constraints.md
```

Regras:

- arquivos canônicos guardam a visao consolidada
- historico diario e append-only
- artefatos aprovados vao para `08_artifacts/`
- o contexto e sincronizado por Git

## O Que Ja Esta Implementado

- onboarding guiado por perguntas
- ativacao por etapa e por pilar
- bootstrap automatico do cliente
- indexacao do contexto
- geracao de jobs iniciais
- regra obrigatoria de growth com `onboarding-media-plan`
- skill oficial `plano-de-midia-make` incorporada ao repositorio
- modulo de Social Media com pipeline versionado, tutorial Meta e skill de copy unificada
- conexao real da Meta para Social com token, Instagram Business Account ID e relatorio mensal
- publicacao do plano aprovado no contexto
- atualizacao automatica de `04_channels/makeads.md`
- sessao conversacional com estado persistido
- proxima pergunta automatica
- interpretacao de aprovacoes em texto comum

## Camada Conversacional

Esta e a parte que resolve o problema da operacao leiga.

O sistema agora tem sessoes de conversa persistidas no control plane.

Isso permite:

- continuar o onboarding sem perder o estado
- entender resposta por texto livre
- fazer a proxima pergunta automaticamente
- bootstrapar sem comando manual
- tratar aprovacao do plano sem terminal

Endpoints internos dessa camada:

- `POST /conversation/message`
- `GET /conversation/sessions/{session_id}`

Skills dessa camada:

- `conversation-intake-router`
- `conversation-next-question`
- `conversation-approval-router`

## API Principal

### Registry

- `GET /registry/agents`
- `GET /registry/skills`

### Onboarding

- `GET /onboarding/project-template`
- `GET /onboarding/stages/{stage}`
- `POST /onboarding/project-plan`
- `POST /onboarding/bootstrap-client`

### Conversa

- `POST /conversation/message`
- `GET /conversation/sessions/{session_id}`

### Integracoes

- `POST /integrations/meta/social/connect`
- `GET /integrations/meta/social/{client_id}`

### Growth

- `POST /growth/media-plan/finalize`

### Social

- `POST /social/reporting/monthly`

### Contexto

- `POST /context/capture`
- `POST /context/index`
- `GET /context/validate`
- `GET /context/clients/{client_id}/blocks`

### Jobs

- `POST /jobs/enqueue`
- `POST /jobs/claim/{worker_id}`
- `POST /jobs/{job_id}/complete`

### Aprovacoes

- `POST /approvals`
- `POST /approvals/{approval_id}/decision`
- `GET /approvals/pending/count`

### Orquestracao

- `GET /orchestrator/daily-plan/{client_id}`

## Arquivos Mais Importantes

- [app.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/app.py)
- [routes.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/api/routes.py)
- [onboarding_service.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/onboarding_service.py)
- [conversation_service.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/conversation_service.py)
- [bootstrap_service.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/bootstrap_service.py)
- [growth_workflow.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/growth_workflow.py)
- [growth_kickoff_service.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/growth_kickoff_service.py)
- [context_service.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/context_service.py)
- [job_queue.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/job_queue.py)
- [registry_data.py](/c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/platform/registry_data.py)

## Modulos Versionados

- [Modulo Social Media](</c:/PRODUÇÕES/CODEX/CEREBRO/docs/SOCIAL_MEDIA_MODULE.md>)
- [Tutorial Meta API para Social](</c:/PRODUÇÕES/CODEX/CEREBRO/docs/SOCIAL_MEDIA_META_API_TUTORIAL.md>)
- [System Prompt da social-copy-engine](</c:/PRODUÇÕES/CODEX/CEREBRO/make-central-brain-core/brain/skills/social/social-copy-engine/system_prompt.md>)

## Setup Tecnico

Esta parte e para manutencao do projeto, nao para uso diario do operador.

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -e .[dev]
python scripts\generate_skill_library.py
python -m pytest
uvicorn brain.app:app --reload
```

## Scripts Internos

Esses scripts existem para testes, bootstrap tecnico e validacao. Nao sao o modo ideal de operacao do time.

- `python scripts\bootstrap_client.py examples\onboarding\cliente-exemplo.json growth`
- `python scripts\finalize_media_plan_kickoff.py <job_id> <arquivo-plano> <arquivo-resumo> growth`

## Estado Atual

Ja esta implementado:

- base do control plane
- contexto versionado
- onboarding progressivo
- kickoff de growth baseado em plano de midia
- camada conversacional com estado

Ainda falta evoluir:

- conectores reais de Meta, Google Ads, GA4, Search Console e MakeCRM
- workers persistentes consumindo fila continuamente
- automacao dos demais jobs especialistas
- observabilidade mais forte

## Validacao

Status atual:

- `21 passed`

Cobertura atual:

- parser de contexto
- validacao de frontmatter
- captura de notas
- onboarding
- bootstrap
- geracao de jobs de growth
- publicacao do plano aprovado
- fluxo conversacional completo
