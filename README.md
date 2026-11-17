<p align="center">
  <img src="assets/banner.png" alt="Mente Afiada — estudo diário com repetição espaçada, gamificação e perguntas geradas por IA" width="100%">
</p>

<p align="center">
  <img alt="Flutter" src="https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white">
  <img alt="Dart" src="https://img.shields.io/badge/Dart-0175C2?logo=dart&logoColor=white">
  <img alt="Firebase" src="https://img.shields.io/badge/Firebase-DD2C00?logo=firebase&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/Scripts-TypeScript-3178C6?logo=typescript&logoColor=white">
  <img alt="Gemini" src="https://img.shields.io/badge/IA-Gemini-8E75B2?logo=googlegemini&logoColor=white">
  <img alt="Android" src="https://img.shields.io/badge/Android-3DDC84?logo=android&logoColor=white">
</p>

# <img src="assets/icone/icone-512.png" width="40" align="top" alt=""> Mente Afiada

App Android para **estudar alguns minutos por dia**. O usuário responde a uma sessão curta de perguntas; as que errou voltam em intervalos crescentes (**repetição espaçada**), cada acerto rende **XP, sequência de dias, conquistas** e pontos na **liga semanal**. Novas perguntas são **geradas por IA**, validadas automaticamente e só entram no app depois de revisadas por um administrador. Para **inglês**, além da múltipla escolha, há exercícios de **audição, ditado e escrita**.

> Esta é a **vitrine pública** do projeto: funcionalidades, arquitetura e telas. O código-fonte fica em um repositório privado.

## Telas

| Login | Início | Sessão de estudo |
| :---: | :---: | :---: |
| <img src="assets/prints/login.png" width="240" alt="Tela de login"> | <img src="assets/prints/inicio.png" width="240" alt="Tela inicial com sequência, meta, nível e liga"> | <img src="assets/prints/sessao.png" width="240" alt="Pergunta respondida com explicação"> |
| **Inglês: ouvir e escolher** | **Inglês: escrita** | **Resultado** |
| <img src="assets/prints/ingles-ouvir.png" width="240" alt="Exercício de audição com botões Ouvir e Devagar"> | <img src="assets/prints/ingles-escrita.png" width="240" alt="Exercício de escrita com a resposta esperada"> | <img src="assets/prints/resultado.png" width="240" alt="Resultado da sessão com XP e bônus"> |
| **Liga semanal** | **Perfil e conquistas** | **Painel do admin** |
| <img src="assets/prints/liga.png" width="240" alt="Ranking da liga semanal"> | <img src="assets/prints/perfil.png" width="240" alt="Perfil com conquistas"> | <img src="assets/prints/admin.png" width="240" alt="Rascunhos gerados pela IA para revisão"> |

## Destaques técnicos

- **Custo zero de infraestrutura:** roda no plano gratuito do Firebase. O que seria Cloud Function virou validação nas **regras do Firestore** e **scripts agendados** no computador do administrador.
- **Anti-trapaça sem servidor:** o app calcula XP, revisão e sequência, e as regras do Firestore conferem cada valor no mesmo lote de escrita: a resposta contra a pergunta real, o XP pela tabela, a transição da caixa de revisão, o bônus de meta uma vez por dia e o XP da liga junto com o XP total.
- **Multiusuário seguro:** dados por usuário nas regras; papel de administrador por *custom claim* do Firebase Auth, nunca por um campo editável.
- **Sem segredos no repositório:** chave da IA só em variável de ambiente da máquina que gera as perguntas; configuração do Firebase e chave da conta de serviço fora do Git.
- **IA com controle de qualidade:** validação automática (4 alternativas distintas, resposta válida, explicação, frases curtas por nível, sem duplicatas parecidas), fila de aprovação do admin, botão "Reportar erro" e teto de custo semanal.
- **Mesma regra em dois lugares, testada uma vez:** a normalização das respostas digitadas existe em Dart (app) e TypeScript (scripts) e as duas são testadas com o mesmo arquivo de casos.

## Funcionalidades

| Área | O que faz |
| --- | --- |
| Conta | Login com e-mail/senha ou Google; exclusão de conta com remoção de todos os dados |
| Primeiro acesso | Escolha de temas, meta diária (5, 10 ou 15 perguntas) e horário do lembrete |
| Estudo | Sessão curta, uma pergunta por vez, feedback imediato e explicação; a pergunta errada volta no fim da sessão |
| Repetição espaçada | Caixas de Leitner: revisão em 1, 3, 7, 14 e 30 dias; até 60% da sessão em revisões |
| Níveis por tema | Nível seguinte liberado com mais de 70% de acerto |
| Inglês | Múltipla escolha, **ouvir e escolher**, **ditado** e **escrita**, com voz do próprio Android, botão "Devagar" e dica |
| Gamificação | XP, níveis, sequência com congelamento, conquistas e ligas Bronze → Diamante |
| Ligas | Grupos semanais de até 30 pessoas; os 7 primeiros sobem, os 5 últimos descem |
| Lembretes | Notificação local no horário escolhido e aviso de sequência em risco (no máximo 2 por dia) |
| Conteúdo por IA | Meta de 50 perguntas por tema, geração semanal e sugestão mensal de temas, sempre como rascunho |
| Administração | Painel no app: aprovar, editar ou rejeitar perguntas (inclusive em lote), tratar denúncias e aprovar temas |

Temas: **SQL**, **Lógica de Programação**, **PHP**, **Inglês para TI**, **IA na prática** e **Matemática**.

## Arquitetura

```mermaid
flowchart LR
    subgraph App["App Android (Flutter + Riverpod)"]
        UI["Telas"]
        TTS["Voz do Android<br/>(audição e ditado)"]
        NOT["Notificações locais"]
    end

    subgraph Firebase["Firebase (plano gratuito)"]
        Auth["Firebase Auth<br/>e-mail e Google"]
        FS[("Cloud Firestore<br/>+ regras de validação")]
    end

    subgraph PC["Computador do admin (Agendador do Windows)"]
        GEN["Geração de perguntas<br/>e temas (TypeScript)"]
        LIG["Fechamento das ligas"]
    end

    IA["Gemini API"]

    UI -- login --> Auth
    UI -- "lê perguntas aprovadas<br/>grava respostas, XP e revisão" --> FS
    UI --> TTS
    UI --> NOT
    GEN -- "Admin SDK" --> FS
    GEN -- "tema, nível, exemplos" --> IA
    LIG -- "Admin SDK" --> FS
```

### Geração de perguntas por IA

```mermaid
sequenceDiagram
    participant S as Agendador do Windows
    participant G as generate-questions.ts
    participant IA as Gemini
    participant DB as Firestore
    participant A as Admin (app)

    S->>G: Domingo, 3h (ou ao ligar o PC)
    G->>DB: Temas e níveis abaixo da meta, uso da semana
    G->>IA: Tema, nível, exemplos e enunciados existentes
    IA-->>G: JSON com perguntas
    G->>G: Valida formato, tamanho e duplicatas
    G->>DB: Salva como rascunho + registro do job
    A->>DB: Aprova, edita ou rejeita no painel
    DB-->>A: Aprovadas aparecem nas sessões
```

## Modelo de dados (Firestore)

| Coleção | Conteúdo |
| --- | --- |
| `topics` | Temas: nome, ícone, cor, status (rascunho/aprovado), origem |
| `questions` | Tipo, enunciado, alternativas ou respostas aceitas, frase falada, explicação, nível, status, denúncias |
| `questions/{id}/reports` | Uma denúncia por usuário |
| `users/{uid}` | Perfil, meta, temas, XP, nível, sequência, liga |
| `users/{uid}/reviews` | Fila da repetição espaçada (caixa e próxima revisão) |
| `users/{uid}/answers` | Histórico de respostas — base para um dashboard de evolução |
| `users/{uid}/achievements` | Conquistas desbloqueadas |
| `leagues/{id}/members` | Grupos semanais por divisão e XP da semana |
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

Nível *n* exige 100 × *n*² XP. Conquistas como "Primeiro passo", "Madrugador" e "Caçador de bugs" (ter uma denúncia de erro confirmada pelo admin).

## Stack

| Camada | Tecnologia |
| --- | --- |
| App | Flutter, Dart, Riverpod, Material 3 |
| Autenticação | Firebase Auth (e-mail e Google) |
| Banco | Cloud Firestore com regras de segurança e validação |
| Scripts | Node.js 22 + TypeScript (Admin SDK), agendados no Windows |
| IA | Gemini API |
| Voz | Text-to-speech do Android (`flutter_tts`) |
| Notificações | `flutter_local_notifications` |
| Testes | `flutter_test`, `node:test` |

## Roteiro

- [x] **Fase 1 — Base:** projeto Flutter, Firebase, login, boas-vindas, regras de segurança e banco inicial de perguntas
- [x] **Fase 2 — Estudo:** sessão de perguntas, gravação das respostas, repetição espaçada e tela de temas
- [x] **Fase 3 — Gamificação:** XP validado pelas regras, níveis, sequência, conquistas, resultado e perfil
- [x] **Fase 4 — Social e lembretes:** ligas semanais, ranking e notificações locais
- [x] **Fase 5 — IA:** geração agendada, painel do admin e denúncias
- [x] **Fase 6 — Perguntas mais difíceis e inglês:** níveis mais exigentes, audição, ditado e escrita
- [ ] Publicação na Google Play (teste fechado) e dashboard de evolução com os dados de `answers`

## Autor

**Fagner Soares** — Desenvolvedor de Sistemas · Salvador, BA
[github.com/fagnersoares](https://github.com/fagnersoares)

O código-fonte é privado; posso apresentá-lo em entrevistas ou sob solicitação.
