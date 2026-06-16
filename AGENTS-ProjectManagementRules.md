## Project Management Rules

> Esta seção é lida por agentes de IA ao gerarem artefatos de gestão,
> tasks, documentação ou specs. Regras marcadas com **DEVE** são
> obrigatórias. Regras marcadas com **NÃO DEVE** são proibições hard.
> Regras marcadas com **PODE** são permissões condicionais.

---

### 1. Nomenclatura de Tasks e Issues

#### 1.1 IDs de tasks (`tasks.md`)

O formato de ID é obrigatório e único por módulo:

| Módulo             | Prefixo    | Exemplo          | Path do `tasks.md`                    |
|--------------------|------------|------------------|---------------------------------------|
| pipeline-ingestao  | `INGEST`   | `INGEST-T001`    | `/specs/pipeline-ingestao/tasks.md`   |
| query-endpoint     | `QUERY`    | `QUERY-T001`     | `/specs/query-endpoint/tasks.md`      |
| feedback-api       | `FEEDBACK` | `FEEDBACK-T001`  | `/specs/feedback-api/tasks.md`        |
| teams-bot          | `BOT`      | `BOT-T001`       | `/specs/teams-bot/tasks.md`           |
| painel-web         | `PAINEL`   | `PAINEL-T001`    | `/specs/painel-web/tasks.md`          |

- **DEVE:** toda task gerada em `tasks.md` receber ID no formato `{PREFIXO}-T{NNN}` com zero-padding de 3 dígitos.
- **NÃO DEVE:** criar IDs fora deste formato ou reutilizar IDs de tasks canceladas.
- **DEVE:** o mesmo ID ser usado no Azure DevOps work item correspondente.

#### 1.2 Formato de título de task

```
[{PREFIXO}-T{NNN}] <verbo no infinitivo> <objeto> — <módulo>
```

Exemplos válidos:
```
[QUERY-T003] Implementar validação de input via Zod — query-endpoint
[INGEST-T007] Criar extrator de tabelas para arquivos XLSX — pipeline-ingestao
[BOT-T001] Configurar Bot Framework SDK no Teams — teams-bot
```

- **NÃO DEVE:** criar task sem verbo de ação no título.
- **NÃO DEVE:** criar task com escopo que cubra mais de 1 dia de trabalho.

#### 1.3 Labels obrigatórias no Azure DevOps e GitHub

Toda task ou issue **DEVE** ter ao menos uma label de cada categoria:

| Categoria  | Labels válidas                                                               |
|------------|------------------------------------------------------------------------------|
| Tipo       | `spec` `task` `bug` `improvement` `question` `change-request`                |
| Módulo     | `pipeline-ingestao` `query-endpoint` `feedback-api` `teams-bot` `painel-web` |
| Status     | `draft` `in-review` `approved` `in-progress` `blocked` `done`                |
| Prioridade | `priority-critical` `priority-high` `priority-normal` `priority-low`         |

- **DEVE:** qualquer issue ou task criada por agente incluir a label `spec` e o módulo correspondente.
- **NÃO DEVE:** criar issue sem label de módulo — impede filtragem no board de tracking.

#### 1.4 Formato de título de Pull Request

```
[TIPO][módulo/artefato][minor|major|criacao] <descrição curta>
```

Exemplos válidos:
```
[SPEC][query-endpoint/requirements][minor] Clarifica VC-03 para perguntas multi-domínio
[SPEC][pipeline-ingestao/plan][major] Adiciona suporte a XLSX com pré-processamento
[CODE][query-endpoint][QUERY-T003] Implementa validação de input via Zod
[FIX][teams-bot][BOT-T005] Corrige timeout na resposta do Bot Framework
```

- **DEVE:** todo PR que altera spec incluir `[SPEC]` no título.
- **DEVE:** todo PR de código referenciar o ID da task (`[QUERY-T003]`) no título.
- **NÃO DEVE:** criar PR sem descrição explicando o que foi feito, por que e como testar.

---

### 2. Documentação de Decisões

#### 2.1 Architecture Decision Records (ADRs)

Toda decisão técnica ou de escopo significativa **DEVE** ser registrada como ADR.

**Path:** `/docs/adr/`  
**Nomenclatura:** `{NNNN}-{slug-da-decisao}.md`

Exemplos:
```
/docs/adr/0001-escolha-azure-openai.md
/docs/adr/0002-estrategia-context-budget.md
/docs/adr/0003-tratamento-docs-contraditorios.md
/docs/adr/0004-azure-ai-search-vs-opensearch.md
```

Formato obrigatório de cada ADR:
```markdown
# ADR-{NNNN} — {Título da decisão}

**Status:** proposta | aceita | depreciada | substituída por ADR-{NNNN}
**Data:** YYYY-MM-DD
**Autores:** {nomes}

## Contexto
## Decisão
## Consequências
## Alternativas Consideradas
```

- **DEVE:** toda decisão que altere arquitetura, modelo de LLM, estratégia de RAG, context budget ou tratamento de documentos gerar um ADR antes de implementar.
- **DEVE:** ADRs com status `aceita` serem referenciados nas specs afetadas no campo `prior-decisions` do `requirements.md`.
- **NÃO DEVE:** agente tomar decisão técnica significativa sem verificar se existe ADR correspondente em `/docs/adr/`.

ADRs existentes e vigentes:
```
ADR-0001 — Escolha do modelo LLM (GPT-4o via Azure OpenAI)
ADR-0002 — Estratégia de context budget (4K system + 8K chunks + 3 turnos)
ADR-0003 — Tratamento de documentos contraditórios (metadado de vigência)
ADR-0004 — Pipeline RAG (Azure AI Search + Azure OpenAI)
```

#### 2.2 Change Requests

Toda mudança em spec com status `em-implementacao` ou posterior **DEVE** gerar um CR.

**Path:** `/docs/change-requests/`  
**Nomenclatura:** `CR-{NNNN}-{slug-do-modulo}.md`

Exemplo:
```
/docs/change-requests/CR-0001-query-endpoint.md
```

- **DEVE:** CR incluir as 4 seções obrigatórias: descrição da mudança, análise de impacto (escopo, prazo, specs afetadas), informações do cliente, e registro do círculo de aprovação.
- **NÃO DEVE:** agente modificar spec com status `aprovada` ou posterior sem CR aberto e aprovado pelo círculo completo (PS → TL → NovaTech → DM).
- **DEVE:** CR aprovado que altere decisão arquitetural também gerar ADR em `/docs/adr/`.

---

### 3. Validation Gates

Nenhuma transição de etapa ocorre sem aprovação humana explícita.  
O agente **NÃO DEVE** avançar automaticamente entre gates.

#### G1 — Aceite de Spec (`Spec → Plan`)

```yaml
gate: G1
nome: Aceite de Spec
transicao: requirements.md aprovado → plan.md pode ser iniciado
aprovadores: [Tech Lead, Product Specialist, Delivery Manager]
todos_devem_aprovar: true
tempo_disponivel: 4h úteis
bloqueio: sprint não inicia sem aprovação dos três
checklist: /docs/checklists/gate-1-aceite-spec.md

pre_condicoes:
  - requirements.md tem status "em-revisao"
  - cabeçalho da spec preenchido: spec, version, status, autor
  - todos os verification criteria são testáveis pelo QA
  - ADRs referenciadas existem em /docs/adr/

se_reprovar:
  - spec retorna para status "rascunho"
  - dono recebe lista de ajustes via comentário no PR
  - sprint não começa até nova aprovação
```

- **NÃO DEVE:** agente iniciar geração de `plan.md` se `requirements.md` não tiver status `aprovada` no cabeçalho.

#### G2 — Validação de Tasks (`Tasks → Implement`)

```yaml
gate: G2
nome: Validacao de Tasks
transicao: tasks.md aprovado → implementação pode iniciar
aprovadores: [Tech Lead]
aprovador_adicional: Dev Sênior (tasks atribuídas ao Dev Pleno)
tempo_disponivel: 2h úteis
bloqueio: nenhum dev inicia sem tasks validadas pelo TL
checklist: /docs/checklists/gate-2-validacao-tasks.md

pre_condicoes:
  - plan.md tem status "aprovada"
  - cada task tem ID no formato {PREFIXO}-T{NNN}
  - cada task tem critério de aceite verificável
  - cada task é concluível em no máximo 1 dia
  - dependências entre tasks mapeadas no Azure DevOps

se_reprovar:
  - tasks específicas são comentadas com ajuste necessário no PR
  - dev reescreve com apoio do Copilot e solicita nova revisão
  - implementação só inicia após nova aprovação do TL
```

- **NÃO DEVE:** agente atribuir task ao Dev Pleno com complexidade acima do perfil sem indicar explicitamente para revisão do TL.

#### G3 — Code Review (`Implement → Merge`)

```yaml
gate: G3
nome: Code Review
transicao: PR aprovado → merge no branch main
aprovadores:
  - Tech Lead: PRs que tocam pipeline RAG, Azure Functions, ADRs, Bicep
  - Dev Sênior: PRs de funcionalidade, bot Teams, painel React
tempo_disponivel: 4h úteis
bloqueio: PR não é mergeado sem aprovação humana
checklist: /docs/checklists/gate-3-code-review.md

pre_condicoes:
  - dev fez self-review com Claude e documentou pontos verificados
  - PR referencia ID da task no título: [QUERY-T003]
  - testes unitários escritos para lógica principal
  - nenhum secret hardcoded no código ou arquivos de config

regras_criticas:
  - context budget respeitado: 4K system + 8K chunks (5x1500t) + 3 turnos
  - metadado de vigência aplicado para docs contraditórios (ADR-0003)
  - TypeScript tipado sem uso excessivo de "any"
  - logs sem exposição de dados sensíveis de usuários

se_reprovar:
  - PR volta ao dev com comentários específicos no GitHub
  - dev corrige, roda novo self-review com Claude, solicita nova revisão
  - ciclo se repete até aprovação — qualidade não é negociável
```

- **DEVE:** todo PR de código gerado por agente incluir comentário `// implements {TASK-ID}` na função ou módulo principal implementado.
- **NÃO DEVE:** agente fazer merge de PR sem aprovação explícita do revisor humano designado.

#### G4 — Quality Gate de RAG (`Review → Deploy`)

```yaml
gate: G4
nome: Quality Gate RAG
transicao: golden dataset aprovado → deploy para HMG ou produção
aprovadores: [QA, Tech Lead]
ambos_devem_aprovar: true
tempo_disponivel: 8h úteis
bloqueio: nenhum deploy sem 100% de aprovação no golden dataset
checklist: /docs/checklists/gate-4-quality-rag.md
golden_dataset: /prompts/eval/golden-queries.json

cenarios_criticos_obrigatorios:
  - alucinacao: assistente não inventa informação ausente na base
  - documento_contradatorio: PROC-042 v1 vs v2 → prioriza versão mais recente (ADR-0003)
  - gap_documental: pergunta sem cobertura → sinaliza "não encontrei"
  - tier_invalido: cliente "Platinum" → redireciona para Gold/Silver/Standard
  - carga_perigosa: restrição POL-001 respeitada → não devolução pelo processo padrão

estado_esperado_base:
  - 847 documentos válidos indexados no Azure AI Search
  - 63 documentos descartados por obsolescência
  - 12 documentos com contradições pendentes: status "pendente-compliance"

se_reprovar:
  - deploy bloqueado
  - QA documenta falhas com classificação: bug-codigo | problema-rag | gap-documental
  - dev corrige → novo ciclo G3 + G4 obrigatório antes de novo deploy
  - após 2 ciclos sem resolução: TL e DM são acionados para decisão
```

- **NÃO DEVE:** agente promover build para HMG ou produção sem registro de aprovação do G4 pelo QA e pelo TL no checklist correspondente.

#### G5 — Aceite de Release (`Deploy → Produção`)

```yaml
gate: G5
nome: Aceite de Release
transicao: G4 aprovado → release para produção e comunicação à NovaTech
aprovadores: [Product Specialist, Delivery Manager]
ambos_devem_aprovar: true
tempo_disponivel: 4h úteis após G4
bloqueio: release não vai para produção sem aprovação do DM
checklist: /docs/checklists/gate-5-aceite-release.md

pre_condicoes:
  - G4 aprovado por QA e TL
  - release notes preparadas em português (linguagem executiva)
  - changelog técnico atualizado em /prompts/prompt-changelog.md se prompt mudou
  - NovaTech comunicada com antecedência sobre janela de deploy
  - nenhum impedimento crítico aberto no Azure DevOps sem responsável

se_reprovar:
  - PS: feature volta para ajuste → novo ciclo G3 obrigatório
  - DM: release adiada → nova data comunicada à NovaTech em até 1h
  - impedimento registrado como crítico no Azure DevOps com responsável e prazo
```

- **NÃO DEVE:** agente gerar comunicado de release para a NovaTech sem aprovação explícita do DM.

---

### 4. Restrições de Comunicação e Geração de Artefatos

#### 4.1 Idioma por tipo de artefato

| Artefato                                    | Idioma    | Justificativa                                    |
|---------------------------------------------|-----------|--------------------------------------------------|
| Código-fonte (`.ts`, `.tsx`)                | Inglês    | Padrão do ecossistema TypeScript/Node            |
| Comentários de código                       | Inglês    | Consistência com o código                        |
| Nomes de variáveis e funções                | Inglês    | Padrão do ecossistema                            |
| Mensagens de commit                         | Inglês    | Padrão git convencional                          |
| ADRs (`/docs/adr/`)                         | Português | Lidos pelo time e pela NovaTech                  |
| Specs (`/specs/`)                           | Português | Lidos pelo cliente e pelo time de negócio        |
| Release notes                               | Português | Comunicação executiva com a NovaTech             |
| Comunicados de status                       | Português | Comunicação com a NovaTech                       |
| Change Requests (`/docs/change-requests/`)  | Português | Assinados pelo cliente                           |
| Checklists de gate                          | Português | Preenchidos por todos os papéis do time          |
| `AGENTS.md`                                 | Português | Lido por todo o time, incluindo não-devs         |
| Mensagens de interface (bot/web)            | Português | Usuário final é atendente da NovaTech            |
| Mensagem "não encontrei"                    | Português | Exibida ao atendente em operação                 |

- **DEVE:** agente gerar toda documentação de gestão (specs, ADRs, CRs, release notes) em português, independente do idioma do prompt recebido.
- **NÃO DEVE:** agente gerar specs ou release notes em inglês, mesmo que solicitado em inglês — o cliente (NovaTech) opera em português.

#### 4.2 Restrições de conteúdo por artefato

- **DEVE:** toda resposta do assistente ao atendente citar a fonte (documento + seção).
- **NÃO DEVE:** assistente gerar resposta sem chunk recuperado do Azure AI Search — sem grounding = sem resposta; retornar mensagem padrão de não encontrado.
- **NÃO DEVE:** assistente inventar informação não presente nos documentos indexados.
- **NÃO DEVE:** spec referenciar decisão técnica que contradiga ADR com status `aceita`.
- **NÃO DEVE:** `tasks.md` ser gerado com base em `plan.md` com status diferente de `aprovada`.

#### 4.3 Restrições de estrutura de arquivos

- **NÃO DEVE:** agente criar pastas de spec fora de `/specs/`.
- **NÃO DEVE:** agente criar arquivos de spec com nome diferente de `requirements.md`, `plan.md` ou `tasks.md`.
- **NÃO DEVE:** agente criar ADR fora de `/docs/adr/`.
- **NÃO DEVE:** agente criar CR fora de `/docs/change-requests/`.
- **NÃO DEVE:** agente modificar `AGENTS.md` sem instrução explícita do Tech Lead ou DM.
- **DEVE:** todo novo módulo de spec ter sua pasta criada com os 3 arquivos vazios antes de qualquer conteúdo ser gerado.

#### 4.4 Restrições de status de spec

- **NÃO DEVE:** agente alterar o campo `status` no cabeçalho de qualquer spec — transição de status é responsabilidade exclusivamente humana.
- **NÃO DEVE:** agente gerar conteúdo de `plan.md` se `requirements.md` do mesmo módulo não tiver status `aprovada`.
- **NÃO DEVE:** agente gerar conteúdo de `tasks.md` se `plan.md` do mesmo módulo não tiver status `aprovada`.
- **DEVE:** agente sinalizar explicitamente quando tentar gerar um artefato cujo pré-requisito de gate não está satisfeito, indicando qual gate precisa ser completado antes de prosseguir.
