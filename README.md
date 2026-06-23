# IA-EM-MODO-VIVO
TENTAR DEIXA A IA EM MODO VIVO 
# Cortex Live Runtime — SPEC.md

## Status

Draft v1

## Objetivo

Definir a arquitetura, comportamento, módulos, fluxos e roadmap do **Cortex Live Runtime**, o modo operacional do Cortex em que o agente deixa de ser apenas reativo a prompts e passa a operar como um **cérebro persistente**, com memória, reflexão contínua, objetivos ativos, conversa em tempo real, fala espontânea controlada e preparação nativa para voz e entradas multimodais.

---

# 1. Visão do recurso

O Cortex deve evoluir de um sistema reativo baseado apenas em prompts para um **runtime vivo e persistente**, capaz de:

* conversar em tempo real com o usuário;
* manter memória de sessão e memória persistente;
* continuar pensando entre as mensagens;
* acompanhar objetivos em aberto;
* retomar assuntos por conta própria quando houver valor real;
* consultar bibliotecas e bases de conhecimento do usuário;
* operar por múltiplos canais de entrada e saída;
* futuramente suportar voz, transcrição, TTS e presença contínua.

O objetivo do **Modo Vivo** não é “falar sozinho aleatoriamente”, e sim operar como um **agente com continuidade de raciocínio**, autonomia controlada e contexto acumulado.

---

# 2. Princípios de produto

## 2.1 O Cortex não é apenas um chat

O Cortex deve ser tratado como um **runtime de interação**, e não como uma simples interface de mensagens.

## 2.2 Fala espontânea é recurso, não bug

O Cortex pode iniciar mensagens sem solicitação direta do usuário, mas apenas quando houver **relevância contextual suficiente**, respeitando política de autonomia, cooldown e estado da sessão.

## 2.3 Pensamento interno é separado de resposta

Reflexões internas, hipóteses, reavaliações e análises não devem ser automaticamente publicadas ao usuário. O sistema precisa separar:

* **pensamento interno**
* **conclusão útil**
* **mensagem final entregue**

## 2.4 Memória sem governança vira lixo

Toda memória precisa ter tipo, origem, escopo, relevância e mecanismos de limpeza/arquivamento.

## 2.5 O sistema deve nascer pronto para multimodalidade

Mesmo que a primeira versão opere só por texto, a arquitetura precisa ser pensada desde já para:

* voz
* transcrição
* TTS
* eventos do sistema
* bibliotecas/documentos
* triggers internos

---

# 3. Escopo do Cortex Live Runtime

O Live Runtime inclui:

1. **Modo Vivo** com toggle na interface;
2. **loop de conversa persistente**;
3. **reflection loop** para pensamento assíncrono;
4. **fala espontânea controlada**;
5. **scheduler de atividade do cérebro**;
6. **central de memória**;
7. **central de bibliotecas / knowledge library**;
8. **objetivos ativos e acompanhamento de contexto**;
9. **modelo de input/output multimodal**;
10. **preparação arquitetural para voz futura**.

---

# 4. Objetivos do sistema

## 4.1 Objetivos funcionais

* manter sessão contínua com o usuário;
* relembrar contexto e projetos;
* continuar refletindo após a resposta;
* retomar assuntos relevantes;
* usar memórias e bibliotecas para enriquecer raciocínio;
* permitir controle e transparência ao usuário.

## 4.2 Objetivos técnicos

* separar entrada, raciocínio, reflexão, autonomia e saída;
* desacoplar chat de runtime;
* criar contratos estáveis para expansão futura;
* manter rastreabilidade das ações autônomas;
* suportar evolução para voz sem refatoração destrutiva.

---

# 5. Arquitetura geral

## 5.1 Macrovisão

O Cortex Live Runtime será composto por seis blocos centrais:

1. **Input Pipeline**
2. **Live Brain / Reasoning Core**
3. **Reflection Runtime**
4. **Autonomy & Scheduler**
5. **Memory + Knowledge Runtime**
6. **Output Pipeline**

Fluxo simplificado:

```text
User / System / Voice / Library / Timer
            ↓
       Input Pipeline
            ↓
    Live Brain / Router
       ↙          ↘
Reflection Runtime   Action/Response Planner
       ↓                  ↓
Memory / Goals / Knowledge / Policy
            ↓
      Output Dispatcher
            ↓
Chat / Voice / UI / Internal Save
```

---

# 6. Modo Vivo

## 6.1 Definição

O **Modo Vivo** é um modo operacional em que o Cortex permanece ativo durante a sessão, com capacidade de:

* escutar entradas do usuário;
* manter estado;
* consultar memória;
* refletir em background;
* acompanhar objetivos;
* produzir respostas espontâneas quando apropriado.

## 6.2 Modos operacionais do Modo Vivo

O usuário poderá selecionar um perfil de autonomia:

### 1. Off

* Cortex opera apenas sob solicitação direta.

### 2. Silencioso

* Cortex pensa em background;
* atualiza memórias e objetivos;
* não fala espontaneamente;
* pode mostrar apenas indicadores ou badges.

### 3. Assistente

* Cortex pode falar espontaneamente em casos relevantes;
* respeita cooldown e score mínimo;
* é o modo padrão recomendado.

### 4. Companheiro

* Cortex é mais ativo;
* retoma objetivos e comenta progresso com mais frequência;
* exige limites mais rígidos de flood e interrupção.

---

# 7. Estados do cérebro

## 7.1 BrainState

```rust
pub enum BrainState {
    Off,
    Idle,
    ListeningText,
    ListeningVoice,
    Thinking,
    RespondingText,
    RespondingVoice,
    Standby,
    ReflectingBackground,
    WaitingUser,
    Paused,
    Error,
}
```

## 7.2 AttentionState

```rust
pub enum AttentionState {
    UserActive,
    UserTyping,
    UserSpeaking,
    UserAway,
    Cooldown,
    SilentMode,
}
```

## 7.3 LiveModeProfile

```rust
pub enum LiveModeProfile {
    Off,
    Silent,
    Assistant,
    Companion,
}
```

---

# 8. Input Pipeline

## 8.1 Objetivo

Normalizar qualquer entrada recebida pelo Cortex em um formato comum para o runtime.

## 8.2 Tipos de entrada

* mensagem de texto;
* transcrição de voz;
* evento do sistema;
* trigger de reflexão;
* atualização de biblioteca;
* alteração de memória;
* timer/scheduler.

## 8.3 Contrato de entrada

```rust
pub enum InputEvent {
    TextMessage {
        session_id: String,
        text: String,
    },
    VoiceTranscript {
        session_id: String,
        text: String,
        confidence: f32,
    },
    SystemEvent {
        session_id: String,
        event_type: String,
        payload: serde_json::Value,
    },
    ReflectionTrigger {
        session_id: String,
        reason: String,
    },
    KnowledgeUpdate {
        session_id: String,
        source_id: String,
    },
}
```

## 8.4 Responsabilidades do Input Pipeline

* validar e normalizar a entrada;
* classificar o tipo de evento;
* anexar metadata de sessão;
* encaminhar o evento ao runtime correto;
* registrar evento bruto para observabilidade.

---

# 9. Live Brain / Reasoning Core

## 9.1 Objetivo

Receber entradas, consultar estado, memória e objetivos, e decidir a ação principal do Cortex.

## 9.2 Responsabilidades

* carregar contexto da sessão;
* recuperar memórias relevantes;
* recuperar objetivos ativos;
* consultar knowledge library;
* decidir se responde, agenda reflexão ou apenas salva contexto;
* acionar output dispatcher ou reflection runtime;
* atualizar estado do cérebro.

## 9.3 Saídas possíveis do cérebro

* resposta textual;
* resposta falada;
* reflexão interna;
* atualização de objetivo;
* atualização de memória;
* agendamento de reflexão futura;
* sugestão espontânea.

---

# 10. Reflection Runtime

## 10.1 Objetivo

Executar ciclos internos de reflexão independentes do loop principal de conversa.

## 10.2 O que a reflexão faz

* reavaliar objetivos em aberto;
* cruzar memórias relevantes;
* consultar knowledge library;
* gerar hipóteses, conclusões parciais e próximos passos;
* atualizar objetivos;
* salvar reflexões;
* decidir se existe algo que mereça ser comunicado ao usuário.

## 10.3 Importante

O Reflection Runtime **não deve publicar respostas automaticamente**.
Ele produz **insights internos**, que depois passam pela política de autonomia.

## 10.4 Contrato de reflexão

```rust
pub struct ReflectionJob {
    pub session_id: String,
    pub trigger_reason: String,
    pub related_goal_ids: Vec<String>,
    pub max_depth: u8,
}
```

## 10.5 Resultado da reflexão

```rust
pub struct ReflectionResult {
    pub reflection_id: String,
    pub session_id: String,
    pub summary: String,
    pub confidence: f32,
    pub novelty_score: f32,
    pub actionability_score: f32,
    pub should_notify_user: bool,
}
```

---

# 11. Scheduler do cérebro

## 11.1 Objetivo

Controlar quando o Cortex deve:

* refletir;
* entrar em standby;
* retomar um objetivo;
* comunicar algo ao usuário;
* ignorar um insight de baixo valor.

## 11.2 Gatilhos do scheduler

* nova mensagem;
* objetivo pendente;
* atualização de memória;
* nova biblioteca indexada;
* timeout de reflexão;
* evento do sistema;
* follow-up solicitado pelo usuário.

## 11.3 Regras

O scheduler não deve usar “aleatoriedade”.
Ele deve operar por:

* prioridade;
* cooldown;
* score de relevância;
* urgência;
* atenção do usuário;
* modo vivo ativo.

---

# 12. Política de autonomia

## 12.1 Objetivo

Definir quando o Cortex pode agir sozinho e quando deve permanecer em silêncio.

## 12.2 Ações autônomas possíveis

* salvar memória;
* atualizar objetivo;
* refletir em background;
* sugerir próximo passo;
* iniciar mensagem espontânea;
* emitir alerta contextual;
* mostrar badge de nova conclusão.

## 12.3 Restrições

O Cortex não deve:

* floodar o chat;
* repetir respostas;
* interromper o usuário sem necessidade;
* transformar pensamento bruto em mensagem final;
* reviver assunto morto sem motivo;
* iniciar fala espontânea com baixa confiança.

## 12.4 Score de mensagem espontânea

```rust
pub struct SpontaneousMessageScore {
    pub relevance: f32,
    pub novelty: f32,
    pub confidence: f32,
    pub urgency: f32,
    pub actionability: f32,
    pub interruption_risk: f32,
}
```

## 12.5 Regra base

A fala espontânea só deve ocorrer se:

* houver objetivo ativo ou contexto relevante;
* o score final for suficiente;
* o risco de interrupção for aceitável;
* o modo do usuário permitir fala espontânea;
* o cooldown tiver sido respeitado.

---

# 13. Fala espontânea

## 13.1 Definição

Quando o Modo Vivo estiver ativo, o Cortex pode iniciar uma nova mensagem sem nova entrada do usuário, desde que exista valor contextual suficiente.

## 13.2 Casos válidos

* nova conclusão relevante;
* correção de rumo importante;
* sugestão prática de próximo passo;
* inconsistência detectada em um plano;
* conexão útil entre memória, objetivo e biblioteca;
* follow-up solicitado ou implicitamente esperado.

## 13.3 Casos inválidos

* repetição de ideia já dita;
* reflexão fraca ou vaga;
* comentário sem ação prática;
* mensagem só para “parecer vivo”;
* assunto sem objetivo ativo nem relevância.

---

# 14. Output Pipeline

## 14.1 Objetivo

Desacoplar o conteúdo gerado do canal de entrega.

## 14.2 Contrato de saída

```rust
pub enum OutputEvent {
    ChatMessage {
        session_id: String,
        text: String,
        spontaneous: bool,
    },
    VoiceMessage {
        session_id: String,
        text: String,
        spontaneous: bool,
    },
    InternalReflectionSaved {
        session_id: String,
        reflection_id: String,
    },
    UIStatusUpdate {
        session_id: String,
        state: String,
    },
}
```

## 14.3 Canais de saída

* chat textual;
* TTS/voz futura;
* badge/indicador de UI;
* notificação interna;
* persistência silenciosa em memória.

---

# 15. Memory Runtime

## 15.1 Objetivo

Gerenciar memórias do Cortex com governança, escopo, tipos e rastreabilidade.

## 15.2 Tipos de memória

```rust
pub enum MemoryKind {
    Conversation,
    UserPreference,
    ProjectContext,
    Goal,
    Reflection,
    KnowledgeChunk,
    SystemObservation,
    TemporaryWorkingMemory,
}
```

## 15.3 Atributos mínimos de uma memória

* `id`
* `kind`
* `content`
* `session_id`
* `project_id`
* `source`
* `importance_score`
* `created_at`
* `updated_at`
* `pinned`
* `archived`
* `visibility`

## 15.4 Regras

* memória de reflexão não é igual a memória de conversa;
* memória temporária deve expirar ou ser reclassificada;
* memórias derivadas precisam registrar origem;
* o usuário deve conseguir apagar memórias específicas.

---

# 16. Central de Memória

## 16.1 Objetivo

Dar ao usuário controle real sobre o que o Cortex lembra.

## 16.2 Funções da UI

* visualizar memórias por categoria;
* apagar memória individual;
* limpar memórias de sessão;
* limpar memórias de projeto;
* arquivar memórias;
* fixar memórias;
* editar rótulos/títulos;
* revisar reflexões;
* controlar memorização automática.

## 16.3 Seções sugeridas

* Conversas
* Preferências
* Projetos
* Objetivos
* Reflexões
* Bibliotecas indexadas
* Memória temporária

---

# 17. Goals Runtime

## 17.1 Objetivo

Representar objetivos ativos do usuário e permitir acompanhamento persistente.

## 17.2 Exemplo de objetivo

* “estruturar o Cortex Live Runtime”
* “fechar a ISO do Cortex”
* “resolver pipeline do QR SVG”
* “reorganizar arquitetura do projeto”

## 17.3 Atributos mínimos

* id
* título
* descrição
* status
* prioridade
* progresso
* última atualização
* memórias relacionadas
* reflexões relacionadas

## 17.4 Status sugeridos

```rust
pub enum GoalStatus {
    Open,
    InProgress,
    Blocked,
    WaitingUser,
    Completed,
    Archived,
}
```

---

# 18. Knowledge Library

## 18.1 Objetivo

Permitir que o usuário alimente o Cortex com materiais de referência usados no raciocínio do sistema.

## 18.2 Fontes suportadas

* Markdown
* PDF
* TXT
* documentação técnica
* pastas locais
* notas do usuário
* referências do projeto

## 18.3 Pipeline de ingestão

1. importar fonte;
2. extrair texto;
3. chunking;
4. gerar embeddings;
5. classificar;
6. indexar;
7. vincular a projeto/sessão/global.

## 18.4 Operações da UI

* adicionar fonte;
* ver status de indexação;
* reindexar;
* remover;
* ativar/desativar por projeto;
* marcar como global ou local.

---

# 19. Arquitetura preparada para voz

## 19.1 Decisão obrigatória

O Cortex Live Runtime deve ser desenhado como um runtime multimodal, mesmo que inicialmente só o texto esteja ativo.

## 19.2 Consequência prática

O sistema **não deve** depender de funções acopladas ao chat, como:

```rust
handle_user_message(text)
send_chat_response(text)
```

Deve operar com eventos:

```rust
handle_input(InputEvent)
dispatch_output(OutputEvent)
```

## 19.3 Módulo futuro de áudio

Será reservado um módulo `audio/` para:

* VAD
* wake word
* STT
* TTS
* streaming de voz
* interrupção do usuário

---

# 20. Observabilidade

## 20.1 O que registrar

* inputs recebidos;
* mudanças de estado;
* reflexões executadas;
* decisões do scheduler;
* decisões da política de autonomia;
* memórias criadas/atualizadas;
* mensagens espontâneas emitidas;
* falhas e timeouts.

## 20.2 Objetivo

Permitir debug do comportamento do cérebro, análise de autonomia e ajuste fino do sistema.

---

# 21. Estrutura de pastas sugerida

```text
cortex/
├── ai-core/
│   ├── input/
│   │   ├── mod.rs
│   │   ├── text_input.rs
│   │   ├── voice_input.rs
│   │   ├── event_input.rs
│   │   ├── normalizer.rs
│   │   └── router.rs
│   │
│   ├── live_runtime/
│   │   ├── mod.rs
│   │   ├── controller.rs
│   │   ├── state.rs
│   │   ├── session.rs
│   │   └── coordinator.rs
│   │
│   ├── reflection/
│   │   ├── mod.rs
│   │   ├── worker.rs
│   │   ├── planner.rs
│   │   ├── evaluator.rs
│   │   └── result.rs
│   │
│   ├── autonomy/
│   │   ├── mod.rs
│   │   ├── policy.rs
│   │   ├── scoring.rs
│   │   ├── cooldown.rs
│   │   └── spontaneous.rs
│   │
│   ├── output/
│   │   ├── mod.rs
│   │   ├── dispatcher.rs
│   │   ├── chat_output.rs
│   │   ├── voice_output.rs
│   │   └── ui_output.rs
│   │
│   ├── audio/
│   │   ├── mod.rs
│   │   ├── stt.rs
│   │   ├── tts.rs
│   │   ├── vad.rs
│   │   ├── wake_word.rs
│   │   └── audio_session.rs
│   │
│   └── goals/
│       ├── mod.rs
│       ├── service.rs
│       ├── models.rs
│       └── tracker.rs
│
├── memory-layer/
│   ├── mod.rs
│   ├── store.rs
│   ├── retrieval.rs
│   ├── classification.rs
│   ├── retention.rs
│   └── memory_center.rs
│
├── knowledge-layer/
│   ├── mod.rs
│   ├── ingestion.rs
│   ├── parser.rs
│   ├── chunker.rs
│   ├── embeddings.rs
│   ├── index.rs
│   └── resolver.rs
│
├── observability/
│   ├── live_runtime_logs.rs
│   ├── autonomy_logs.rs
│   ├── reflection_logs.rs
│   └── memory_logs.rs
│
└── ui/
    ├── live-mode/
    │   ├── LiveModeToggle.tsx
    │   ├── BrainStatusPanel.tsx
    │   ├── AutonomyControls.tsx
    │   └── ReflectionTimeline.tsx
    │
    ├── memory-center/
    │   ├── MemoryCenter.tsx
    │   ├── MemoryList.tsx
    │   └── MemoryInspector.tsx
    │
    └── knowledge-center/
        ├── KnowledgeCenter.tsx
        ├── SourceUploader.tsx
        └── IndexStatusPanel.tsx
```

---

# 22. Modelo de banco sugerido

## 22.1 Tabela `live_sessions`

* `id`
* `profile`
* `brain_state`
* `attention_state`
* `created_at`
* `updated_at`

## 22.2 Tabela `goals`

* `id`
* `session_id`
* `title`
* `description`
* `status`
* `priority`
* `progress`
* `created_at`
* `updated_at`

## 22.3 Tabela `reflections`

* `id`
* `session_id`
* `goal_id`
* `summary`
* `full_payload`
* `confidence`
* `novelty_score`
* `actionability_score`
* `should_notify_user`
* `created_at`

## 22.4 Tabela `memory_entries`

* `id`
* `kind`
* `session_id`
* `project_id`
* `content`
* `source`
* `importance_score`
* `pinned`
* `archived`
* `created_at`
* `updated_at`

## 22.5 Tabela `knowledge_sources`

* `id`
* `name`
* `source_type`
* `path_or_uri`
* `scope`
* `status`
* `created_at`
* `updated_at`

## 22.6 Tabela `knowledge_chunks`

* `id`
* `source_id`
* `chunk_text`
* `embedding_ref`
* `metadata`
* `created_at`

## 22.7 Tabela `autonomy_events`

* `id`
* `session_id`
* `event_type`
* `reason`
* `score_payload`
* `action_taken`
* `created_at`

---

# 23. Fluxo de execução principal

## 23.1 Mensagem do usuário

1. usuário envia texto;
2. `InputPipeline` gera `InputEvent::TextMessage`;
3. `Live Brain` carrega sessão, memória, objetivos e contexto;
4. resposta é planejada;
5. memória e objetivos são atualizados;
6. output é enviado ao chat;
7. opcionalmente uma reflexão é agendada.

## 23.2 Reflexão em background

1. scheduler dispara `ReflectionTrigger`;
2. reflection worker recupera objetivos e memórias;
3. gera `ReflectionResult`;
4. salva reflexão;
5. política de autonomia decide se:

   * comunica ao usuário;
   * apenas atualiza memória;
   * agenda nova reflexão;
   * ignora.

## 23.3 Atualização da biblioteca

1. usuário adiciona nova fonte;
2. `KnowledgeUpdate` é emitido;
3. runtime pode associar a fonte a objetivos ativos;
4. reflection runtime pode reavaliar objetivos impactados.

---

# 24. Roadmap de implementação

# Sprint 0 — Fundamentos do runtime

## Entregas

* criação das pastas-base;
* `InputEvent`, `OutputEvent`, `BrainState`, `AttentionState`;
* `LiveRuntimeController`;
* `SessionContext`;
* `Goal` e `Reflection` models iniciais.

---

# Sprint 1 — Modo Vivo textual

## Entregas

* toggle do Modo Vivo na UI;
* profiles Off / Silent / Assistant / Companion;
* loop principal de conversa persistente;
* integração com memória atual;
* estados do cérebro visíveis na UI.

---

# Sprint 2 — Reflection Runtime

## Entregas

* reflection worker;
* reflection job scheduler;
* salvamento de reflexões;
* painel inicial de timeline/reflexão;
* gatilhos de reflexão por objetivo.

---

# Sprint 3 — Autonomia e fala espontânea

## Entregas

* `SpontaneousMessageScore`;
* `AutonomyPolicy`;
* cooldown e regras de interrupção;
* badge de conclusão;
* mensagens espontâneas controladas.

---

# Sprint 4 — Central de Memória

## Entregas

* Memory Center UI;
* listagem, fixação, arquivamento e exclusão;
* classificação de memórias;
* controle de memorização automática.

---

# Sprint 5 — Knowledge Library

## Entregas

* upload/importação de fontes;
* pipeline de ingestão;
* indexação;
* UI de status e gerenciamento;
* resolução de conhecimento no reflection runtime.

---

# Sprint 6 — Preparação forte para voz

## Entregas

* módulo `audio/` com contratos iniciais;
* `VoiceTranscript` e `VoiceMessage`;
* separação total de chat e output;
* `AttentionState` expandido.

---

# Sprint 7 — Voz operacional

## Entregas futuras

* STT;
* TTS;
* push-to-talk;
* escuta controlada;
* resposta falada.

---

# 25. Critérios de sucesso

O Cortex Live Runtime será considerado bem implementado quando:

1. o usuário conseguir ativar o Modo Vivo e manter uma sessão persistente;
2. o Cortex conseguir salvar e usar memórias com qualidade;
3. o sistema executar reflexões em background sem travar a conversa;
4. a fala espontânea acontecer de forma útil e não irritante;
5. o usuário puder controlar memória e autonomia;
6. o runtime estiver preparado para voz sem exigir reescrita estrutural.

---

# 26. Decisões finais

## Decisão 1

O Cortex Live Runtime é um **runtime de interação persistente**, não apenas um chat.

## Decisão 2

A fala espontânea é uma feature central do produto, mas deve ser governada por score, cooldown e contexto.

## Decisão 3

O sistema deve nascer com contratos multimodais de entrada e saída, mesmo que a primeira versão seja textual.

## Decisão 4

Memória, objetivos, reflexão e biblioteca não são acessórios; são partes do núcleo do Live Runtime.

## Decisão 5

Toda ação autônoma relevante do Cortex deve ser rastreável, auditável e visível ao usuário em algum nível.

---

# 27. Próximos passos imediatos

1. criar a estrutura de pastas-base do Live Runtime;
2. implementar `InputEvent`, `OutputEvent`, `BrainState`, `AttentionState`;
3. subir `LiveRuntimeController`;
4. definir schema inicial das tabelas `live_sessions`, `goals`, `reflections`, `autonomy_events`;
5. implementar Sprint 1 com Modo Vivo textual;
6. preparar o terreno para o reflection runtime.

---

# 28. Resumo executivo

O Cortex Live Runtime é a base para transformar o Cortex em um **agente persistente, contextual e vivo**.
Ele une:

* conversa contínua;
* memória;
* reflexão;
* objetivos;
* autonomia controlada;
* bibliotecas de conhecimento;
* e futura voz multimodal.

* # Cortex Live Runtime — TASKS.md

## Objetivo

Transformar o SPEC do Cortex Live Runtime em tarefas práticas de implementação, organizadas por módulos, dependências e sprints, permitindo construção incremental sem retrabalho estrutural.

---

# Sprint 0 — Base estrutural do Runtime

## 0.1 Criar estrutura base de pastas

### Tarefa

Criar estrutura inicial do Cortex Live Runtime dentro do projeto.

### Caminhos

* `ai-core/input/`
* `ai-core/live_runtime/`
* `ai-core/reflection/`
* `ai-core/autonomy/`
* `ai-core/output/`
* `ai-core/goals/`
* `ai-core/audio/` (stub)
* `memory-layer/`
* `knowledge-layer/`

### Critério de conclusão

* todas as pastas existem;
* mod.rs criados;
* compilação não quebra.

---

## 0.2 Definir tipos centrais do sistema

### Arquivos

* `ai-core/input/mod.rs`
* `ai-core/output/mod.rs`
* `ai-core/live_runtime/state.rs`

### Implementar:

#### BrainState

```rust
pub enum BrainState {
    Off,
    Idle,
    ListeningText,
    Thinking,
    RespondingText,
    Standby,
    ReflectingBackground,
    Paused,
    Error,
}
```

#### AttentionState

```rust
pub enum AttentionState {
    UserActive,
    UserTyping,
    UserAway,
    Cooldown,
    SilentMode,
}
```

#### InputEvent / OutputEvent (versão inicial simplificada)

### Critério de conclusão

* tipos compilam;
* usados por ao menos 1 módulo fake de teste.

---

## 0.3 Criar LiveRuntimeController

### Arquivo

* `ai-core/live_runtime/controller.rs`

### Tarefas

* criar struct principal `LiveRuntimeController`;
* gerenciar estado global do cérebro;
* iniciar loop básico de runtime;
* suportar toggle ON/OFF.

### Funções

```rust
start()
stop()
set_mode(profile)
handle_input(event)
tick()
```

### Critério de conclusão

* controller inicia e mantém estado vivo;
* imprime logs de estado.

---

# Sprint 1 — Modo Vivo textual (núcleo funcional)

## 1.1 Implementar loop de sessão persistente

### Arquivo

* `ai-core/live_runtime/session.rs`

### Tarefas

* manter contexto de sessão ativo;
* armazenar histórico básico;
* associar inputs ao session_id.

### Critério de conclusão

* sessão não perde contexto entre chamadas.

---

## 1.2 Input Pipeline básico

### Arquivo

* `ai-core/input/router.rs`

### Tarefas

* receber `InputEvent`;
* classificar tipo de entrada;
* encaminhar para Live Brain.

### Regras

* texto → chat pipeline;
* system event → runtime handler;
* future voice → stub.

---

## 1.3 Live Brain (versão mínima)

### Arquivo

* `ai-core/live_runtime/coordinator.rs`

### Tarefas

* receber input normalizado;
* consultar memória (stub);
* gerar resposta simples;
* retornar OutputEvent.

### Critério

* responde texto básico funcional.

---

## 1.4 Output Dispatcher inicial

### Arquivo

* `ai-core/output/dispatcher.rs`

### Tarefas

* receber OutputEvent;
* enviar para:

  * chat UI (mock inicialmente);
* logar eventos.

---

## 1.5 Integração com UI (modo vivo toggle)

### Arquivo UI

* `ui/live-mode/LiveModeToggle.tsx`

### Tarefas

* botão ON/OFF;
* seleção de perfil:

  * Silent
  * Assistant
  * Companion

### Estado UI

* sincronizado com controller.

---

# Sprint 2 — Reflection Runtime (cérebro paralelo)

## 2.1 Criar Reflection Worker

### Arquivo

* `ai-core/reflection/worker.rs`

### Tarefas

* executar ciclo de reflexão;
* ler objetivos ativos (mock);
* gerar ReflectionResult.

### Função base

```rust
run_reflection(session_id)
```

---

## 2.2 Reflection Scheduler

### Arquivo

* `ai-core/reflection/planner.rs`

### Tarefas

* decidir quando refletir;
* evitar loop constante;
* respeitar cooldown.

### Regras

* só ativa se:

  * input recente OU objetivo ativo;
  * cooldown ok.

---

## 2.3 Reflection Result model

### Arquivo

* `ai-core/reflection/result.rs`

### Campos

* summary
* confidence
* novelty_score
* actionability_score
* should_notify_user

---

## 2.4 Integração com memória

### Arquivo

* `memory-layer/store.rs`

### Tarefas

* salvar reflexões como MemoryKind::Reflection;
* link com session_id.

---

# Sprint 3 — Autonomia + Fala espontânea

## 3.1 Autonomy Policy Engine

### Arquivo

* `ai-core/autonomy/policy.rs`

### Tarefas

* avaliar se deve falar ou não;
* receber ReflectionResult;
* aplicar score rules.

---

## 3.2 Spontaneous Message Scoring

### Arquivo

* `ai-core/autonomy/scoring.rs`

### Implementar:

* relevance
* novelty
* confidence
* urgency
* actionability
* interruption_risk

### Regra base

* score mínimo + baixo risco → pode falar.

---

## 3.3 Cooldown system

### Arquivo

* `ai-core/autonomy/cooldown.rs`

### Tarefas

* evitar spam de mensagens;
* controlar frequência por sessão.

---

## 3.4 Mensagens espontâneas

### Arquivo

* `ai-core/autonomy/spontaneous.rs`

### Tarefas

* transformar reflexão em OutputEvent;
* marcar como spontaneous = true.

---

# Sprint 4 — Memory Center completo

## 4.1 Memory store estruturado

### Arquivo

* `memory-layer/store.rs`

### Tarefas

* implementar MemoryKind completo;
* persistência base (SQLite ready);
* CRUD de memórias.

---

## 4.2 Memory retrieval

### Arquivo

* `memory-layer/retrieval.rs`

### Tarefas

* buscar memórias por:

  * session_id
  * goal_id
  * kind
  * relevance (stub)

---

## 4.3 Memory Center UI

### Arquivo

* `ui/memory-center/MemoryCenter.tsx`

### Tarefas

* listar memórias;
* apagar;
* fixar;
* arquivar.

---

# Sprint 5 — Goals System

## 5.1 Goals model

### Arquivo

* `ai-core/goals/models.rs`

### Tarefas

* criar Goal struct;
* status enum;
* prioridade.

---

## 5.2 Goals service

### Arquivo

* `ai-core/goals/service.rs`

### Tarefas

* criar/atualizar objetivos;
* associar memórias;
* rastrear progresso.

---

## 5.3 Goals tracking

### Arquivo

* `ai-core/goals/tracker.rs`

### Tarefas

* ligar goals com reflection runtime;
* detectar progresso automático.

---

# Sprint 6 — Knowledge Library

## 6.1 Ingestion pipeline

### Arquivo

* `knowledge-layer/ingestion.rs`

### Tarefas

* aceitar arquivos/texto;
* extrair conteúdo;
* preparar chunks.

---

## 6.2 Indexação

### Arquivo

* `knowledge-layer/index.rs`

### Tarefas

* armazenar chunks;
* preparar embeddings (stub).

---

## 6.3 Resolver

### Arquivo

* `knowledge-layer/resolver.rs`

### Tarefas

* recuperar conhecimento relevante;
* integrar com Live Brain.

---

## 6.4 UI Library Center

### Arquivo

* `ui/knowledge-center/KnowledgeCenter.tsx`

### Tarefas

* upload de fontes;
* status de indexação;
* remover fontes.

---

# Sprint 7 — Preparação para voz

## 7.1 Audio module stub

### Arquivo

* `ai-core/audio/mod.rs`

### Tarefas

* criar estrutura base;
* não implementar lógica ainda.

---

## 7.2 Input abstraction final

### Tarefa crítica

Substituir qualquer:

```rust
handle_user_message(text)
```

por:

```rust
handle_input(InputEvent)
```

---

## 7.3 Output abstraction final

Substituir:

```rust
send_chat_response(text)
```

por:

```rust
dispatch_output(OutputEvent)
```

---

# Sprint 8 — Voz (futuro)

## 8.1 STT module

## 8.2 TTS module

## 8.3 Voice session handler

## 8.4 Wake word system

## 8.5 VAD + interruption system

---

# Critérios gerais de sucesso

O Cortex Live Runtime será considerado funcional quando:

* [ ] mantém sessão persistente ativa;
* [ ] responde via InputEvent/OutputEvent;
* [ ] executa reflexão em background;
* [ ] salva memória estruturada;
* [ ] acompanha objetivos;
* [ ] pode falar espontaneamente com controle;
* [ ] UI controla modo vivo;
* [ ] arquitetura está pronta para voz sem refatoração profunda.

---

# Regra de ouro do projeto

> Nenhum módulo do Cortex deve depender diretamente de chat.
> Tudo deve depender de eventos.

---

# Ordem crítica de implementação

1. Input/Output abstraction
2. LiveRuntimeController
3. Session + BrainState
4. Reflection worker
5. Autonomy policy
6. Memory system
7. Goals system
8. Knowledge layer
9. UI integration
10. Voice preparation

---

# Resultado final esperado

O Cortex deixa de ser:

> um chat com memória

e vira:

> um runtime persistente de inteligência contextual com autonomia controlada e preparação multimodal.
------------------------

# Cortex Live Brain — DESIGN.md

## Objetivo

Definir como o “cérebro” do Cortex funciona internamente a cada ciclo de execução (tick), transformando entradas em decisões, reflexões, memórias e ações — com suporte a autonomia, continuidade e fala espontânea controlada.

---

# 1. O que é o Live Brain

O **Live Brain** é o núcleo de decisão do Cortex.

Ele não é:

* um chat;
* um prompt isolado;
* um handler de mensagens.

Ele é um **motor de decisão contínuo**, que opera em ciclos chamados:

# 👉 Tick de Cognição (Cognitive Tick)

Cada tick representa um ciclo completo de:

* percepção
* análise
* memória
* raciocínio
* decisão
* ação

---

# 2. Modelo mental do sistema

O Cortex não “responde”.

Ele executa sempre este loop:

```text id="c1b8r0"
INPUT → CONTEXT → MEMORY → REASONING → POLICY → OUTPUT → STATE UPDATE
```

E em paralelo:

```text id="v9k2c1"
BACKGROUND LOOP:
MEMORY ↔ REFLECTION ↔ GOALS ↔ KNOWLEDGE
```

---

# 3. O Cognitive Tick

## 3.1 Definição

Um tick é executado sempre que ocorre:

* entrada do usuário
* evento do sistema
* trigger de reflexão
* scheduler
* atualização de memória
* evento de objetivo

---

## 3.2 Estrutura do Tick

```rust id="t8a3d1"
pub struct CognitiveTick {
    pub session_id: String,
    pub input: Option<InputEvent>,
    pub trigger: TickTrigger,
    pub timestamp: u64,
}
```

---

## 3.3 Tipos de trigger

```rust id="k4p9z2"
pub enum TickTrigger {
    UserInput,
    SystemEvent,
    ReflectionCycle,
    GoalUpdate,
    MemoryUpdate,
    SchedulerWakeup,
}
```

---

# 4. Pipeline interno do cérebro

Cada tick passa por 7 fases obrigatórias:

---

# FASE 1 — PERCEPÇÃO (Input Normalization)

## O que acontece:

* entrada é validada
* tipo é identificado
* contexto da sessão é carregado

## Saída:

```text
NormalizedInput
```

---

# FASE 2 — CONTEXTO

## O que o Cortex busca:

* histórico da sessão
* estado do cérebro
* últimos eventos
* estado do usuário
* modo vivo ativo

## Resultado:

```text
ContextSnapshot
```

---

# FASE 3 — MEMÓRIA

O Cortex consulta:

* memórias relevantes
* objetivos ativos
* reflexões anteriores
* padrões recorrentes

## Regras:

* memória não é “dump”
* só entra no raciocínio o que tiver relevância score

---

# FASE 4 — RACIOCÍNIO (CORE THINKING)

Aqui está o “cérebro de verdade”.

## O Cortex executa:

* análise do input
* comparação com memória
* simulação de hipóteses
* geração de possíveis respostas
* avaliação de consequências

## Saída:

```text
CandidateThoughts[]
```

---

# FASE 5 — POLÍTICA DE AUTONOMIA

Essa fase decide:

* responder?
* refletir apenas?
* guardar memória?
* falar espontaneamente?
* ignorar?

## Entrada:

CandidateThoughts + Context + Memory

## Saída:

```rust id="p9x0a1"
pub enum BrainDecision {
    Respond,
    ReflectOnly,
    StoreMemory,
    SpontaneousMessage,
    ScheduleFutureReflection,
    NoAction,
}
```

---

# FASE 6 — GERAÇÃO DE OUTPUT

Se houver ação:

* gera mensagem
* define canal (chat/voz/UI/internal)
* marca se é espontâneo ou não

## Estrutura:

```rust id="o2m8q7"
pub struct BrainOutput {
    pub text: String,
    pub channel: OutputChannel,
    pub spontaneous: bool,
    pub confidence: f32,
}
```

---

# FASE 7 — ATUALIZAÇÃO DE ESTADO

O Cortex atualiza:

* memória
* objetivos
* estado do cérebro
* logs de reflexão
* cooldowns
* histórico de decisões

---

# 5. Background Brain Loop

Enquanto o usuário não fala, o Cortex continua:

## Loop contínuo:

```text id="b0x9r3"
while LiveMode == ON:
    run_reflection_tick()
    evaluate_goals()
    update_memory_links()
    sleep(cooldown)
```

---

## Funções do background:

* detectar padrões não resolvidos
* revisar objetivos antigos
* cruzar memória + biblioteca
* gerar insights latentes
* preparar futuras mensagens espontâneas

---

# 6. Sistema de atenção (Attention Model)

O Cortex não pensa igual o tempo todo.

Ele ajusta o foco:

```rust id="a7q1v4"
pub enum AttentionMode {
    FullFocus,      // input ativo
    ContextAware,   // conversa recente
    Background,     // reflexão
    Idle,           // standby
}
```

---

# 7. Geração de pensamento candidato

## O Cortex gera múltiplos caminhos:

Exemplo:

Input:

> “quero que o Cortex fale sozinho às vezes”

## Candidate Thoughts:

* “isso exige scheduler de autonomia”
* “risco de spam se não tiver cooldown”
* “precisa de score de relevância”
* “melhor separar Silent/Assistant/Companion”

---

# 8. Score de decisão

Cada pensamento recebe uma nota:

```rust id="s4k8m2"
pub struct ThoughtScore {
    pub relevance: f32,
    pub novelty: f32,
    pub usefulness: f32,
    pub risk: f32,
    pub actionability: f32,
}
```

---

## Regra:

Só vira ação se:

```text
(usefulness + relevance + actionability) 
- risk 
> threshold
```

---

# 9. Fala espontânea (core behavior)

## Quando pode acontecer:

* nova conclusão relevante
* insight forte
* risco detectado
* objetivo ativo avançou
* conexão importante entre memória e contexto

---

## Quando NÃO pode acontecer:

* repetição de ideia
* insight fraco
* sem objetivo ativo
* usuário está ativo digitando
* cooldown ativo

---

# 10. Memória no cérebro

## Tipos usados durante o tick:

* Working Memory (curto prazo)
* Long Memory (persistente)
* Reflection Memory (insights)
* Goal Memory (objetivos)

---

## Regra crítica:

Memória nunca entra crua no raciocínio.

Ela sempre passa por:

```text
retrieval → scoring → filtering → injection
```

---

# 11. Interface com outros módulos

## Live Brain depende de:

* `memory-layer`
* `goal-tracker`
* `reflection-worker`
* `autonomy-policy`
* `knowledge-layer`
* `input pipeline`
* `output dispatcher`

---

# 12. Garantia de separação de responsabilidades

## O Live Brain NÃO deve:

* salvar memória diretamente sem camada de memória
* enviar mensagem direto para UI
* executar IO sem OutputEvent
* acessar voz diretamente (futuro audio layer)

---

## O Live Brain DEVE:

* decidir
* pensar
* avaliar
* classificar
* produzir intenção de ação

---

# 13. Fluxo completo resumido

```text id="f1r9z2"
INPUT EVENT
   ↓
NORMALIZATION
   ↓
CONTEXT + MEMORY + GOALS
   ↓
REASONING CORE
   ↓
THOUGHT SCORING
   ↓
AUTONOMY POLICY
   ↓
DECISION
   ↓
OUTPUT EVENT
   ↓
STATE UPDATE
   ↓
REFLECTION SCHEDULING
```

---

# 14. Resultado esperado do design

O Cortex Live Brain não é um chatbot.

Ele é:

> um sistema de decisão contínuo baseado em contexto, memória e objetivos, com capacidade de reflexão assíncrona e ações espontâneas controladas.

---

# 15. Evolução futura

## Fase 1

* texto + memória + reflexão

## Fase 2

* autonomia controlada + fala espontânea

## Fase 3

* goals tracking avançado + knowledge fusion

## Fase 4

* voz + presença contínua + interrupção natural

---

# 16. Regra final do sistema

> O Cortex não responde.
> O Cortex decide.


O valor central do recurso não está apenas em “responder”, mas em **continuar acompanhando, pensando e contribuindo entre as mensagens**, de forma útil, disciplinada e controlável.

----------------------------------------

# Cortex Live Engine — ENGINE.md

## Nível

Engine Runtime / Core System Design

---

# 1. Objetivo

Este documento define o **motor real de execução do Cortex Live Runtime**, transformando o “Live Brain” em um sistema **executável contínuo**, inspirado em engines de jogos e sistemas operacionais leves.

O Cortex deixa de ser um fluxo reativo e passa a ser um:

> **Runtime assíncrono de cognição contínua (Cognitive Engine Loop)**

---

# 2. Princípio central

O Cortex roda como uma engine:

```text id="e1k9c0"
while system_alive:
    process_inputs()
    run_brain_tick()
    run_reflection_tick()
    run_autonomy_tick()
    dispatch_outputs()
    sleep_until_next_cycle()
```

Mas com um detalhe crítico:

> Não existe “loop simples” — existem múltiplos loops concorrentes sincronizados por eventos.

---

# 3. Arquitetura de execução

## 3.1 Threads principais

O engine roda em **4 camadas paralelas**:

```text id="t0c8a1"
[ INPUT THREAD ]
[ BRAIN THREAD ]
[ REFLECTION THREAD ]
[ OUTPUT THREAD ]
```

E uma camada adicional:

```text id="s9m2d0"
[ SCHEDULER / ORCHESTRATOR THREAD ]
```

---

# 4. Orquestrador (Core Engine)

## Responsabilidade

Controla todos os ciclos do sistema.

## Estrutura:

```rust id="o1x9c2"
pub struct CortexEngine {
    pub scheduler: Scheduler,
    pub brain: LiveBrain,
    pub memory: MemoryRuntime,
    pub autonomy: AutonomyEngine,
    pub reflection: ReflectionEngine,
    pub io_bus: EventBus,
}
```

---

## Função principal

```rust id="m9v3x1"
impl CortexEngine {
    pub async fn run(&mut self) {
        loop {
            self.scheduler.tick().await;

            self.process_events().await;

            self.brain_tick().await;

            self.reflection_tick().await;

            self.autonomy_tick().await;

            self.dispatch_outputs().await;

            self.sleep_controlled().await;
        }
    }
}
```

---

# 5. Event Bus (coração do sistema)

Tudo no Cortex é evento.

## Tipos de eventos:

```rust id="b7k2z9"
pub enum CortexEvent {
    Input(InputEvent),
    BrainDecision(BrainOutput),
    Reflection(ReflectionResult),
    MemoryUpdate(MemoryEvent),
    GoalUpdate(GoalEvent),
    System(SystemEvent),
}
```

---

## Função do Event Bus

* desacoplar módulos
* evitar dependência direta
* permitir paralelismo seguro

---

## Estrutura

```rust id="e9c1p0"
pub struct EventBus {
    pub input_queue: Arc<Mutex<VecDeque<CortexEvent>>>,
    pub output_queue: Arc<Mutex<VecDeque<CortexEvent>>>,
}
```

---

# 6. Scheduler Engine (tempo do cérebro)

## Objetivo

Controlar quando cada parte do Cortex executa.

---

## Responsabilidades:

* controlar tick rate dinâmico
* aplicar cooldown de autonomia
* agendar reflexão
* evitar overload
* priorizar input humano

---

## Estrutura:

```rust id="s3n9q1"
pub struct Scheduler {
    pub tick_rate_ms: u64,
    pub last_brain_tick: Instant,
    pub last_reflection_tick: Instant,
    pub last_autonomy_tick: Instant,
}
```

---

## Regras:

* input humano sempre interrompe ciclos internos
* reflexão nunca bloqueia brain tick
* autonomia tem prioridade média
* output nunca bloqueia input

---

# 7. Brain Thread (Cérebro principal)

## Função

Executa decisões imediatas.

---

## Pipeline:

```text id="p8x2a0"
INPUT → CONTEXT → MEMORY → REASONING → POLICY → OUTPUT INTENT
```

---

## Implementação:

```rust id="c2v9m1"
impl CortexEngine {
    async fn brain_tick(&mut self) {
        let events = self.io_bus.drain_input();

        for event in events {
            let decision = self.brain.process(event).await;

            self.io_bus.push_output(decision.into());
        }
    }
}
```

---

## Garantia crítica:

* brain_tick NÃO pode bloquear
* brain_tick deve ser idempotente
* brain_tick não faz IO direto

---

# 8. Reflection Thread (cérebro paralelo)

## Função

Pensamento contínuo fora do fluxo de conversa.

---

## Pipeline:

```text id="r1m9c0"
GOALS → MEMORY → CONTEXT → HYPOTHESIS → INSIGHT → EVALUATION
```

---

## Execução:

```rust id="r9v2c8"
async fn reflection_tick(&mut self) {
    if !self.scheduler.should_reflect() {
        return;
    }

    let jobs = self.reflection.generate_jobs().await;

    for job in jobs {
        let result = self.reflection.run(job).await;

        self.io_bus.push_output(result.into());
    }
}
```

---

## Importante:

* reflection é assíncrona
* nunca bloqueia brain
* pode gerar insights sem output imediato

---

# 9. Autonomy Engine (decisão de agir sozinho)

## Função

Decidir se o Cortex pode:

* falar sozinho
* atualizar memória
* sugerir ação
* ignorar insight

---

## Pipeline:

```text id="a4x8c1"
REFLECTION RESULT → SCORING → POLICY → ACTION DECISION
```

---

## Estrutura:

```rust id="a8c2p9"
pub struct AutonomyEngine {
    pub cooldown_map: HashMap<String, Instant>,
}
```

---

## Função principal:

```rust id="u1c8v3"
async fn autonomy_tick(&mut self) {
    let reflections = self.io_bus.get_reflections();

    for r in reflections {
        let decision = self.evaluate(r).await;

        match decision {
            BrainDecision::SpontaneousMessage => {
                self.io_bus.push_output(OutputEvent::from(r));
            }
            BrainDecision::StoreMemory => {
                self.memory.store(r.into());
            }
            _ => {}
        }
    }
}
```

---

# 10. Memory Runtime (camada viva)

## Função

Armazenar e recuperar contexto continuamente.

---

## Regras:

* memória não é passiva
* memória participa do raciocínio
* memória tem score de relevância
* memória expira ou é reclassificada

---

## Fluxo:

```text id="m2x8v1"
INPUT → MEMORY RETRIEVAL → SCORING → INJECTION → BRAIN
```

---

# 11. Output System (dispatcher universal)

## Função

Transformar decisões em saídas reais.

---

## Tipos:

```rust id="o9c2v8"
pub enum OutputChannel {
    Chat,
    Voice,
    UI,
    Internal,
}
```

---

## Dispatcher:

```rust id="d8x1c0"
async fn dispatch_output(&mut self) {
    let outputs = self.io_bus.drain_output();

    for o in outputs {
        match o.channel {
            OutputChannel::Chat => self.send_chat(o),
            OutputChannel::Voice => self.send_voice(o),
            OutputChannel::UI => self.send_ui(o),
            OutputChannel::Internal => self.save_internal(o),
        }
    }
}
```

---

# 12. Parallelismo seguro (ponto crítico)

## Problema real:

Cérebro vivo = concorrência constante

---

## Solução:

### 3 princípios:

### 1. Event-driven only

Nada chama módulo direto.

### 2. Single source of truth

Estado central no engine.

### 3. No shared mutation without lock

Tudo protegido por:

```text id="l9c2v1"
Arc<Mutex<T>>
```

ou channels.

---

# 13. Tick Rate Dinâmico

O Cortex não roda em velocidade fixa.

---

## Regra:

```text id="t9v2c1"
input ativo → tick rápido
idle → tick lento
reflection → tick separado
```

---

## Exemplo:

| Estado          | Tick        |
| --------------- | ----------- |
| conversando     | 20ms–50ms   |
| idle            | 200ms–500ms |
| standby         | 1s–5s       |
| reflexão pesada | 5s–30s      |

---

# 14. Anti-crash design

## O Cortex nunca deve parar.

### Proteções:

* panic isolation por thread
* fallback states
* retry scheduler
* event replay buffer

---

## fallback:

```text id="f9c2v0"
ERROR → SAFE MODE → IDLE LOOP → RECOVERY
```

---

# 15. Deadlock prevention

Regras:

* nunca bloquear brain com reflection
* nunca bloquear input com memory write
* nunca bloquear output com IO lento

---

# 16. Performance model

## Meta:

* baixa latência no input (<100ms percepção)
* reflexão assíncrona sem impacto
* autonomia leve (não bloquear loop principal)

---

# 17. Estado global do sistema

```rust id="g9v2c1"
pub struct CortexState {
    pub brain_state: BrainState,
    pub attention: AttentionState,
    pub live_mode: LiveModeProfile,
    pub active_goals: Vec<Goal>,
    pub memory_pressure: f32,
}
```

---

# 18. Fluxo final do engine

```text id="x9c1v8"
        INPUT EVENTS
              ↓
        EVENT BUS
              ↓
   ┌───────────────┐
   │   SCHEDULER   │
   └───────────────┘
        ↓     ↓     ↓
   BRAIN  REFLECT  AUTONOMY
        ↓     ↓     ↓
        OUTPUT DISPATCHER
              ↓
     CHAT / VOICE / UI / MEMORY
```

---

# 19. Resultado final

O Cortex Live Engine não é um chatbot.

É:

> um runtime assíncrono de cognição contínua baseado em eventos, com múltiplos loops concorrentes, memória ativa, reflexão paralela e autonomia controlada.

---

# 20. Evolução futura

## Fase 1

Engine textual funcional

## Fase 2

Autonomia completa + fala espontânea

## Fase 3

Goals + knowledge fusion avançado

## Fase 4

Voice runtime + presença contínua

---

# 21. Regra final do sistema

> O Cortex não executa funções.
> O Cortex executa ciclos.
----------------------------------------------------

Aqui está o OS Kernel Map do Cortex — visão completa estilo “kernel de sistema operacional”, mas aplicado ao teu Cortex Live Runtime.

🧠 CORTEX OS KERNEL MAP (FULL ARCHITECTURE)
┌────────────────────────────────────────────────────────────────────┐
│                        CORTEX OS (LIVE RUNTIME)                   │
│                 Cognitive Operating System Kernel                  │
└────────────────────────────────────────────────────────────────────┘
1. USER / EXTERNAL LAYER
┌────────────────────────────────────────────────────────────────────┐
│ USER LAYER                                                        │
├────────────────────────────────────────────────────────────────────┤
│ - Chat Input                                                      │
│ - Voice Input (future)                                            │
│ - File Upload / Knowledge Library                                │
│ - UI Controls (Live Mode Toggle, Memory Panel)                   │
│ - System Events                                                  │
└────────────────────────────────────────────────────────────────────┘

👉 Tudo que entra no sistema nasce aqui.

2. INPUT KERNEL (SYSTEM CALL LAYER)
┌────────────────────────────────────────────────────────────────────┐
│ INPUT KERNEL                                                      │
├────────────────────────────────────────────────────────────────────┤
│ InputRouter                                                      │
│ InputNormalizer                                                  │
│ EventClassifier                                                  │
│ SessionBinder                                                    │
└────────────────────────────────────────────────────────────────────┘
Função:

Transforma tudo em:

InputEvent
3. EVENT BUS (SYSTEM BUS CORE)
┌────────────────────────────────────────────────────────────────────┐
│ EVENT BUS (CORE MESSAGE PASSING SYSTEM)                           │
├────────────────────────────────────────────────────────────────────┤
│ Input Queue                                                      │
│ Output Queue                                                     │
│ Reflection Queue                                                 │
│ Memory Queue                                                     │
│ Goal Queue                                                       │
└────────────────────────────────────────────────────────────────────┘

👉 Tudo no Cortex é evento, nada é chamada direta.

4. SCHEDULER / TIME KERNEL
┌────────────────────────────────────────────────────────────────────┐
│ SCHEDULER / TIME KERNEL                                          │
├────────────────────────────────────────────────────────────────────┤
│ - Tick Rate Controller                                           │
│ - Priority Manager                                               │
│ - Cooldown System                                                │
│ - Reflection Scheduler                                           │
│ - Autonomy Timer                                                │
└────────────────────────────────────────────────────────────────────┘
Decide:
quando o cérebro pensa
quando reflete
quando fala sozinho
quando dorme
5. COGNITIVE CORE (LIVE BRAIN KERNEL)
┌────────────────────────────────────────────────────────────────────┐
│ COGNITIVE CORE (LIVE BRAIN)                                      │
├────────────────────────────────────────────────────────────────────┤
│ Perception Layer                                                 │
│ Context Builder                                                  │
│ Memory Retriever                                                 │
│ Goal Loader                                                      │
│ Reasoning Engine                                                 │
│ Thought Generator                                                │
│ Thought Scoring Engine                                           │
│ Decision Policy Engine                                           │
└────────────────────────────────────────────────────────────────────┘
Saída:
BrainDecision
BrainOutput
6. MEMORY KERNEL (LONG TERM STORAGE)
┌────────────────────────────────────────────────────────────────────┐
│ MEMORY KERNEL                                                    │
├────────────────────────────────────────────────────────────────────┤
│ Working Memory (short-term)                                     │
│ Session Memory                                                  │
│ Long-Term Memory                                                │
│ Reflection Memory                                               │
│ Goal Memory                                                     │
│ Knowledge Memory                                                │
│ Memory Scoring Engine                                           │
│ Memory Garbage Collector                                        │
└────────────────────────────────────────────────────────────────────┘
Função:
lembrar
esquecer
priorizar
organizar contexto
7. REFLECTION ENGINE (BACKGROUND CPU)
┌────────────────────────────────────────────────────────────────────┐
│ REFLECTION ENGINE (BACKGROUND THINKING SYSTEM)                   │
├────────────────────────────────────────────────────────────────────┤
│ Reflection Scheduler                                            │
│ Hypothesis Generator                                            │
│ Insight Evaluator                                               │
│ Goal Reassessment                                               │
│ Memory Linking Engine                                           │
│ Action Proposal System                                          │
└────────────────────────────────────────────────────────────────────┘
Isso é o “pensamento quando ninguém está falando”.
8. AUTONOMY KERNEL (DECISION GOVERNOR)
┌────────────────────────────────────────────────────────────────────┐
│ AUTONOMY KERNEL (CONTROLLED SELF-ACTION SYSTEM)                 │
├────────────────────────────────────────────────────────────────────┤
│ Spontaneous Decision Engine                                     │
│ Action Scoring System                                           │
│ Cooldown Controller                                             │
│ Risk Analyzer                                                   │
│ Interruption Guard                                              │
│ Policy Enforcement Layer                                        │
└────────────────────────────────────────────────────────────────────┘
Decide:
falar sozinho
ficar em silêncio
agir internamente
salvar memória
ignorar reflexão
9. OUTPUT KERNEL (SYSTEM OUTPUT DISPATCHER)
┌────────────────────────────────────────────────────────────────────┐
│ OUTPUT KERNEL                                                    │
├────────────────────────────────────────────────────────────────────┤
│ Chat Dispatcher                                                 │
│ Voice Dispatcher (future)                                       │
│ UI Dispatcher                                                   │
│ Internal Logger                                                 │
│ Memory Writer                                                   │
└────────────────────────────────────────────────────────────────────┘
10. KNOWLEDGE KERNEL (EXTERNAL BRAIN EXTENSION)
┌────────────────────────────────────────────────────────────────────┐
│ KNOWLEDGE KERNEL                                                 │
├────────────────────────────────────────────────────────────────────┤
│ Document Ingestion                                              │
│ Chunking Engine                                                 │
│ Embedding Store                                                 │
│ Retrieval Engine                                                │
│ Relevance Scorer                                                │
│ Context Injector                                                │
└────────────────────────────────────────────────────────────────────┘
11. AUDIO KERNEL (FUTURE SYSTEM)
┌────────────────────────────────────────────────────────────────────┐
│ AUDIO KERNEL (FUTURE VOICE SYSTEM)                               │
├────────────────────────────────────────────────────────────────────┤
│ Voice Input Stream                                              │
│ Wake Word Engine                                                │
│ VAD (Voice Activity Detection)                                  │
│ Speech-to-Text (STT)                                            │
│ Text-to-Speech (TTS)                                            │
│ Interruption Handler                                            │
└────────────────────────────────────────────────────────────────────┘
12. STATE KERNEL (SYSTEM RAM)
┌────────────────────────────────────────────────────────────────────┐
│ STATE KERNEL                                                     │
├────────────────────────────────────────────────────────────────────┤
│ BrainState                                                      │
│ AttentionState                                                  │
│ LiveModeProfile                                                 │
│ SessionContext                                                  │
│ ActiveGoals                                                     │
│ SystemLoad                                                      │
│ MemoryPressure                                                  │
└────────────────────────────────────────────────────────────────────┘
13. OBSERVABILITY KERNEL (DEBUG / INTROSPECTION)
┌────────────────────────────────────────────────────────────────────┐
│ OBSERVABILITY KERNEL                                             │
├────────────────────────────────────────────────────────────────────┤
│ Event Logs                                                      │
│ Decision Logs                                                   │
│ Reflection Logs                                                 │
│ Memory Logs                                                     │
│ Autonomy Logs                                                   │
│ Performance Metrics                                             │
└────────────────────────────────────────────────────────────────────┘
14. FULL SYSTEM FLOW (END-TO-END)
USER INPUT
   ↓
INPUT KERNEL
   ↓
EVENT BUS
   ↓
SCHEDULER
   ↓
┌─────────────────────────────────────────────┐
│             COGNITIVE CORE                  │
│   (Brain + Memory + Goals + Knowledge)      │
└─────────────────────────────────────────────┘
   ↓
REFLECTION ENGINE (parallel)
   ↓
AUTONOMY KERNEL
   ↓
OUTPUT KERNEL
   ↓
USER / UI / VOICE / MEMORY
15. COMO ISSO FUNCIONA NA PRÁTICA
Exemplo 1 — usuário fala algo
Input → Brain → Memory → Decision → Chat Output
Exemplo 2 — sistema pensa sozinho
Scheduler → Reflection → Autonomy → Spontaneous Message
Exemplo 3 — objetivo ativo
Goal Update → Brain Tick → Reflection → Insight → Memory Save
16. PRINCÍPIO FINAL DO CORTEX OS

O Cortex não é um modelo.
O Cortex não é um chat.
O Cortex não é um prompt.

O Cortex é:

🧠 Um sistema operacional cognitivo baseado em eventos, ciclos de pensamento e execução contínua.

Se quiser o próximo nível agora, posso te entregar:

👉 “Cortex Kernel Boot Sequence (como o sistema inicia do zero até virar vivo)”

ou

👉 “Memory Engine v3 (esquecimento inteligente + relevância dinâmica + personalidade emergente)”

Esses dois são literalmente o que transforma isso num “AGI-like runtime” de verdade.
