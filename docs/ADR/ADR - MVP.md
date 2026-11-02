# Architecture Decision Records (ADRs) --- Nocthar

**Status:** Proposed
**Scope:** MVP
**Date:** 2026-11-02

Este documento registra as decisões arquiteturais propostas para o
modelo de domínio do sistema de batalha de Nocthar. Decisões ainda não
confirmadas estão marcadas como **Proposed**.

------------------------------------------------------------------------

## ADR-001 --- Separar o domínio da batalha da plataforma e da renderização

**Status:** Proposed

### Contexto

O jogo utiliza SDL3 para funcionalidades de plataforma e OpenGL para
renderização. Entretanto, as regras de batalha devem permanecer
independentes dessas tecnologias para facilitar testes, portabilidade e
possíveis mudanças futuras de plataforma.

### Decisão

O núcleo do jogo será independente de SDL3, OpenGL, câmera, animações,
dispositivos de entrada e APIs específicas de plataforma.

As camadas externas serão responsáveis por:

-   Converter entradas do jogador em comandos.
-   Apresentar o estado da batalha.
-   Reagir aos eventos produzidos pelo domínio.

O domínio receberá comandos e atualizará o estado da batalha.

### Consequências

-   A lógica de batalha poderá ser testada sem inicializar recursos
    gráficos.
-   A renderização não será responsável por regras de gameplay.
-   A integração com Web e dispositivos mobile ficará isolada nas
    camadas externas.

------------------------------------------------------------------------

## ADR-002 --- Utilizar Battle como raiz do domínio da batalha

**Status:** Proposed

### Contexto

A batalha coordena participantes, grid, timeline, ações e condições de
encerramento. É necessário definir um ponto central responsável pela
consistência do estado do combate.

### Decisão proposta

A classe `Battle` será a raiz do domínio da batalha (*Aggregate Root*).

Responsabilidades:

-   Inicializar a batalha.
-   Controlar seu estado geral.
-   Coordenar a execução das ações.
-   Verificar condições de vitória e derrota.
-   Encerrar e reiniciar o combate.

A `Battle` não implementará diretamente as regras internas de
movimentação, timeline ou resolução de dano. Ela coordenará os sistemas
responsáveis por essas operações.

### Consequências

-   O fluxo da batalha terá um ponto central de coordenação.
-   Os subsistemas poderão manter responsabilidades específicas.
-   Será necessário definir posteriormente como a `Battle` controla o
    acesso e as alterações aos seus dados.

------------------------------------------------------------------------

## ADR-003 --- Representar participantes por composição

**Status:** Proposed

### Contexto

Nocthar possui participantes com características distintas, incluindo
unidades controladas pelo jogador, inimigos e bosses. Um boss pode
possuir mecânicas especiais, como múltiplos tiles, regiões vulneráveis e
fases.

Criar uma hierarquia rígida de classes para cada tipo de participante
pode aumentar o acoplamento e dificultar a adição de novas mecânicas.

### Decisão proposta

Todos os participantes serão representados por uma estrutura comum
`Participant`. Características especiais serão adicionadas por
composição, utilizando componentes tipados independentes.

### Consequências

-   Bosses e unidades comuns compartilham uma representação fundamental.
-   Mecânicas especiais podem ser adicionadas sem criar subclasses para
    cada variação.
-   O sistema de componentes precisará definir como componentes são
    armazenados, consultados e atualizados.
-   A implementação concreta do sistema de componentes permanece fora
    desta decisão.

------------------------------------------------------------------------

## ADR-004 --- Manter o Grid como sistema lógico independente

**Status:** Proposed

### Contexto

As batalhas acontecem em um grid lógico, mas são apresentadas
visualmente em perspectiva isométrica. A câmera pode alternar entre
quatro ângulos sem alterar as posições lógicas.

### Decisão

O `Grid` será responsável exclusivamente pela representação espacial
lógica da batalha.

Responsabilidades:

-   Representar as dimensões do mapa.
-   Identificar tiles transitáveis e bloqueados.
-   Controlar a ocupação dos tiles.
-   Validar movimentações.
-   Consultar quais participantes ocupam determinadas posições.

A posição será representada por `GridPosition`.

O Grid não terá conhecimento sobre projeção isométrica, sprites, câmera
ou renderização.

### Consequências

-   A rotação da câmera não modifica o estado da batalha.
-   A lógica espacial poderá ser testada sem renderização.
-   A conversão entre coordenadas do grid e coordenadas de tela ficará
    na camada de apresentação.

------------------------------------------------------------------------

## ADR-005 --- Separar o agendamento da execução de ações

**Status:** Proposed

### Contexto

A timeline dinâmica determina qual participante poderá agir em seguida.
Entretanto, determinar o próximo participante e executar sua ação são
responsabilidades diferentes.

### Decisão proposta

O sistema `Timeline` será responsável apenas por determinar a ordem e o
instante das oportunidades de ação.

Cada entrada da timeline (`TimelineEntry`) manterá, no mínimo:

-   Identificador do participante.
-   Progresso acumulado.
-   Frequência ajustada.
-   Prioridade.
-   Estado de participação na timeline.

A frequência ajustada seguirá o modelo em [Dynamic-Timeline.md](docs/Mechanics/Dynamic-Timeline.md)

A `Timeline` não executará comandos nem aplicará regras de movimentação,
dano ou habilidades.

A `Battle` coordenará o recebimento, a validação e a execução das ações.

### Consequências

-   A timeline poderá ser testada independentemente das regras das
    ações.
-   Alterações de atributos poderão atualizar a frequência sem reiniciar
    o progresso acumulado.
-   O sistema de agendamento não ficará acoplado aos tipos de ação
    disponíveis.

------------------------------------------------------------------------

## ADR-006 --- Separar Commands de Events

**Status:** Proposed

### Contexto

O sistema precisa distinguir uma ação solicitada de um acontecimento
efetivamente produzido pela batalha. Uma solicitação pode ser inválida
e, portanto, não produzir uma alteração no estado.

### Decisão proposta

Utilizar dois conceitos distintos:

**Commands** representam intenções ou solicitações de execução.

Exemplos:

-   `MoveCommand`
-   `AttackCommand`
-   `UseAbilityCommand`
-   `InteractCommand`

**Events** representam acontecimentos que efetivamente ocorreram.

Exemplos:

-   `ParticipantMoved`
-   `DamageReceived`
-   `ParticipantDefeated`
-   `BossPhaseChanged`
-   `BattleEnded`

Um comando deve passar por validação antes de alterar o estado. Comandos
rejeitados não devem produzir eventos que indiquem uma alteração que não
aconteceu.

### Consequências

-   A intenção de executar uma ação fica separada de seu resultado.
-   Interface e renderização poderão reagir aos eventos sem controlar a
    lógica.
-   Eventos poderão apoiar depuração, testes e registro de batalhas.
-   Será necessário definir posteriormente se os eventos serão
    armazenados, publicados em fila ou consumidos diretamente.

------------------------------------------------------------------------

## ADR-007 --- Utilizar identificadores estáveis para participantes

**Status:** Proposed

### Contexto

Participantes são referenciados por diferentes sistemas, incluindo
Battle, Grid, Timeline e Events. Essas referências não devem depender de
endereços de memória ou da posição de objetos em contêineres.

### Decisão proposta

Utilizar um tipo dedicado `ParticipantId`, com valores inteiros
sequenciais gerados deterministicamente durante a criação dos
participantes da batalha.

### Consequências

-   Referências entre sistemas serão explícitas e estáveis durante a
    batalha.
-   O sistema não dependerá de ponteiros para identificar participantes.
-   A geração dos IDs deverá seguir uma ordem determinística.
-   A persistência e a unicidade dos IDs entre diferentes batalhas
    poderão ser definidas posteriormente, caso necessário.

------------------------------------------------------------------------

## ADR-008 --- Utilizar Commands tipados para representar ações

**Status:** Proposed

### Contexto

As ações disponíveis possuem parâmetros e regras diferentes. Uma
movimentação exige um destino; um ataque exige um alvo; uma interação
pode exigir um elemento do cenário.

Uma estrutura genérica com vários campos opcionais pode permitir
combinações inválidas e concentrar condicionais em um único executor.

### Decisão proposta

Representar cada solicitação por um Command tipado, mantendo uma
interface ou mecanismo comum de execução.

Exemplos:

-   `MoveCommand`
-   `AttackCommand`
-   `UseAbilityCommand`
-   `InteractCommand`

Cada tipo terá os parâmetros necessários para sua operação e será
validado pelo domínio antes da execução.

### Consequências

-   Os parâmetros de cada ação ficam explícitos.
-   Combinações inválidas de parâmetros podem ser evitadas.
-   Novos tipos de ação podem ser adicionados sem transformar uma
    estrutura genérica em um conjunto crescente de campos opcionais.
-   Será necessário definir posteriormente o mecanismo comum de despacho
    e execução dos comandos.

------------------------------------------------------------------------

## ADR-009 --- Preservar determinismo no domínio da batalha

**Status:** Proposed

### Contexto

O mesmo estado inicial, mapa, participantes, sequência de comandos e
sequência de valores aleatórios deve produzir o mesmo resultado. Isso
facilita testes, reprodução de bugs e uma possível implementação futura
de multiplayer.

### Decisão

As operações do domínio devem ser determinísticas.

Para isso:

-   Empates na timeline devem utilizar regras explícitas de prioridade.
-   Empates de prioridade devem utilizar `ParticipantId` como critério
    final.
-   Valores aleatórios devem ser fornecidos por uma fonte controlável e
    reproduzível.
-   A lógica do domínio não deve depender de relógios, estados globais
    ou APIs de plataforma.
-   A ordem de iteração de contêineres não ordenados não deve determinar
    resultados de gameplay.

### Consequências

-   Testes poderão reproduzir cenários com precisão.
-   Bugs poderão ser investigados utilizando os mesmos comandos e
    valores aleatórios.
-   O sistema terá uma base mais adequada para registro e reprodução de
    batalhas.

------------------------------------------------------------------------

## ADR-010 --- Limitar o escopo inicial do domínio ao necessário para o MVP

**Status:** Proposed

### Contexto

O sistema prevê mecânicas complexas de bosses, mas implementá-las
antecipadamente pode aumentar a complexidade antes de validar o combate
principal.

### Decisão proposta

O primeiro modelo de domínio deverá conter somente:

-   `Battle`
-   `Participant`
-   `ParticipantId`
-   `ParticipantAttributes`
-   `ParticipantState`
-   `Grid`
-   `GridPosition`
-   `Timeline`
-   `TimelineEntry`
-   Commands
-   Events

Componentes especiais de bosses serão adicionados conforme as
necessidades do primeiro encontro jogável.

### Consequências

-   O primeiro protótipo terá um escopo controlado.
-   A arquitetura será validada com mecânicas reais antes de receber
    abstrações adicionais.
-   Novos componentes poderão surgir a partir de requisitos concretos do
    MVP.

------------------------------------------------------------------------

## Resumo das decisões

| ADR | Decisão | Status |
|---|---|---|
| ADR-001 | Separar domínio, plataforma e renderização | Proposed |
| ADR-002 | Utilizar `Battle` como raiz do domínio | Proposed |
| ADR-003 | Representar participantes por composição | Proposed |
| ADR-004 | Manter o Grid independente da apresentação | Proposed |
| ADR-005 | Separar agendamento e execução de ações | Proposed |
| ADR-006 | Separar Commands de Events | Proposed |
| ADR-007 | Utilizar IDs inteiros sequenciais | Proposed |
| ADR-008 | Utilizar Commands tipados | Proposed |
| ADR-009 | Preservar determinismo | Proposed |
| ADR-010 | Limitar o domínio inicial ao MVP | Proposed |

## Próximos passos

1. Revisar e confirmar as decisões propostas.
2. Definir as interfaces públicas dos sistemas principais.
3. Definir a organização dos diretórios e arquivos C++.
4. Implementar o primeiro núcleo de batalha.
5. Criar testes automatizados para Grid, Timeline e execução de ações.
