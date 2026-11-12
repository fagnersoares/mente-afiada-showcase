<p align="center">
  <img src="assets/banner.svg" alt="Mente Afiada" width="100%">
</p>

<p align="center">
  <img alt="Flutter" src="https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white">
  <img alt="Dart" src="https://img.shields.io/badge/Dart-0175C2?logo=dart&logoColor=white">
  <img alt="Firebase" src="https://img.shields.io/badge/Firebase-DD2C00?logo=firebase&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/Cloud%20Functions-TypeScript-3178C6?logo=typescript&logoColor=white">
  <img alt="Android" src="https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white">
  <img alt="Status" src="https://img.shields.io/badge/status-em%20desenvolvimento-FF7A1A">
</p>

# Mente Afiada

App Android de **estudo diário com perguntas de múltipla escolha**. O usuário estuda alguns minutos por dia; as perguntas que errou voltam em intervalos crescentes (**repetição espaçada**), cada acerto rende **XP, sequência de dias e conquistas**, e há **ligas semanais** com ranking. Novas perguntas e temas são **gerados por IA** em agendamento semanal e só entram no app depois de revisados por um administrador.

> Esta é a **vitrine pública** do projeto: documentação, arquitetura e telas. O código-fonte fica em um repositório privado.

## Destaques técnicos

- **Multiusuário seguro desde o início:** regras do Firestore por usuário; papel de administrador por *custom claims*, nunca por um campo editável.
- **Anti-trapaça:** o app nunca calcula XP. Uma Cloud Function valida cada resposta, soma o XP e agenda a revisão; os campos de jogo são bloqueados para o cliente nas regras.
- **Sem segredos no código:** a chave da IA fica só no Secret Manager e é usada apenas pelas Cloud Functions; arquivos de configuração do Firebase ficam fora do Git.
- **IA com controle de qualidade:** validação automática (4 alternativas distintas, resposta entre 0 e 3, explicação, sem duplicatas parecidas), fila de aprovação do admin, botão "Reportar erro" e teto de custo semanal.
- **Dados para análise:** cada resposta é registrada para alimentar um dashboard de evolução (Firebase Analytics + BigQuery).

## Funcionalidades

| Área | O que faz |
| --- | --- |
| Conta | Login com e-mail/senha ou Google; exclusão de conta com remoção de todos os dados |
| Primeiro acesso | Escolha de temas, meta diária (5, 10 ou 15 perguntas) e horário do lembrete |
| Estudo | Sessão curta, uma pergunta por vez, feedback verde/vermelho e explicação após responder |
| Repetição espaçada | Sistema de caixas de Leitner: 1, 3, 7, 14 e 30 dias |
| Gamificação | XP, níveis, sequência de dias com "congelamento", conquistas e ligas Bronze → Diamante |
| Lembretes | Notificação no horário escolhido e aviso de sequência em risco (no máximo 2 por dia) |
| Conteúdo por IA | Geração semanal de perguntas e sugestão mensal de temas, sempre como rascunho |
| Administração | Fila para aprovar, editar ou rejeitar perguntas e tratar denúncias |

Temas iniciais: **SQL**, **Lógica de Programação**, **PHP** e **Inglês para TI**, com um banco inicial de 52 perguntas.

## Arquitetura

```mermaid
flowchart LR
    subgraph App["App Android (Flutter + Riverpod)"]
        UI["Telas"]
    end

    subgraph Firebase
        Auth["Firebase Auth<br/>e-mail e Google"]
        FS[("Cloud Firestore")]
        CF["Cloud Functions<br/>Node.js + TypeScript"]
        SCH["Cloud Scheduler"]
        FCM["Cloud Messaging"]
        SM["Secret Manager"]
    end

    IA["API de modelo de linguagem"]
    BQ[("BigQuery<br/>dashboard futuro")]

    UI -- login --> Auth
    UI -- "lê perguntas aprovadas<br/>grava respostas" --> FS
    FS -- "nova resposta" --> CF
    CF -- "XP, nível, sequência,<br/>revisão" --> FS
    SCH -- "agenda semanal/mensal" --> CF
    CF -- "chave da API" --- SM
    CF -- "gera perguntas" --> IA
    CF -- lembretes --> FCM
    FCM --> UI
    FS -. exportação .-> BQ
```

### Geração de perguntas por IA

```mermaid
sequenceDiagram
    participant S as Cloud Scheduler
    participant F as generateQuestions
    participant IA as Modelo de IA
    participant DB as Firestore
    participant A as Admin

    S->>F: Domingo, 3h
    F->>DB: Temas e níveis com menos perguntas
    F->>IA: Tema, nível, exemplos e enunciados existentes
    IA-->>F: JSON com perguntas
    F->>F: Valida formato e descarta duplicatas
    F->>DB: Salva como rascunho + registro do job
    A->>DB: Aprova, edita ou rejeita
    DB-->>A: Aprovadas aparecem no app
```

## Modelo de dados (Firestore)

| Coleção | Conteúdo |
| --- | --- |
| `topics` | Temas: nome, ícone, cor, status (rascunho/aprovado), origem |
| `questions` | Enunciado, 4 alternativas, índice da correta, explicação, nível, status, estatísticas, denúncias |
| `users/{uid}` | Perfil, meta, temas, XP, nível, sequência, liga |
| `users/{uid}/reviews` | Fila da repetição espaçada (caixa e próxima revisão) |
| `users/{uid}/answers` | Histórico de respostas — base do dashboard |
| `users/{uid}/achievements` | Conquistas desbloqueadas |
| `leagues` | Grupos semanais de até 30 pessoas por divisão |
| `generationJobs` | Auditoria e controle de custo da geração por IA |

## Gamificação

| Ação | XP |
| --- | --- |
| Acerto em pergunta nova | 10 |
| Acerto em revisão | 15 |
| Erro (leu a explicação) | 2 |
| Meta do dia batida | +30 |
| Sessão perfeita | +20 |
| Bônus de sequência | +10% por semana, até +50% |

Nível *n* exige 100 × *n*² XP. Faixas: Iniciante, Aprendiz, Praticante, Especialista e Mestre.

## Stack

| Camada | Tecnologia |
| --- | --- |
| App | Flutter, Dart, Riverpod, Material 3 |
| Autenticação | Firebase Auth |
| Banco | Cloud Firestore com regras por usuário |
| Servidor | Cloud Functions (Node.js 22 + TypeScript) |
| Agendamento | Cloud Scheduler |
| Notificações | Firebase Cloud Messaging |
| IA | API de modelo de linguagem, só no servidor |
| Análise | Firebase Analytics + BigQuery |
| Testes | `flutter_test`, `node:test` |

## Roteiro

- [x] **Fase 1 — Base:** projeto Flutter, Firebase, login, boas-vindas, regras de segurança e importação do banco inicial
- [ ] **Fase 2 — Estudo:** sessão de perguntas, gravação das respostas, repetição espaçada e tela de temas
- [ ] **Fase 3 — Gamificação:** XP por Cloud Function, níveis, sequência, conquistas, resultado e perfil
- [ ] **Fase 4 — Social e lembretes:** ligas semanais, ranking e notificações
- [ ] **Fase 5 — IA:** geração agendada, painel do admin e denúncias
- [ ] Publicação na Google Play (teste fechado) e dashboard de evolução com os dados de `answers`

## Telas

As capturas de tela serão adicionadas aqui conforme cada fase for concluída.

<!--
| Login | Boas-vindas | Início |
| --- | --- | --- |
| <img src="assets/prints/login.png" width="240"> | <img src="assets/prints/boas-vindas.png" width="240"> | <img src="assets/prints/inicio.png" width="240"> |
-->

## Autor

**Fagner Soares** — Desenvolvedor de Sistemas · Salvador, BA
[github.com/fagnersoares](https://github.com/fagnersoares)

O código-fonte é privado; posso apresentá-lo em entrevistas ou sob solicitação.
