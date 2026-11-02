# Nocthar *(nome temporário)*

> Um jogo tático isométrico episódico, focado em batalhas dinâmicas e confrontos contra bosses baseados em estratégia e puzzle.

## Visão Geral

**Nocthar** é um jogo tático desenvolvido em C++.

O jogo acontece em mapas baseados em grid e apresentados em uma perspectiva isométrica 2D. As batalhas utilizam uma **timeline dinâmica**, em vez de rodadas tradicionais, onde a frequência das ações de cada participante é determinada por seus atributos.

O principal foco da experiência são os confrontos contra grandes bosses. Essas batalhas combinam posicionamento tático com mecânicas de puzzle, exigindo que o jogador se movimente estrategicamente, identifique vulnerabilidades e interaja com elementos específicos do campo de batalha.

## Conceitos Principais

### Timeline Dinâmica

O jogo não utiliza rodadas fixas nas quais todos os participantes agem uma vez.

Cada participante recebe oportunidades para agir de acordo com sua frequência efetiva de ações.

Essa frequência é influenciada por:

- **Velocidade** — determina com que frequência um participante pode agir.
- **Periculosidade** — representa o nível de ameaça de participantes especiais, como bosses e sub-bosses.

Conceitualmente:

```text
Frequência Efetiva = Velocidade × Periculosidade
```

Esses valores podem mudar durante uma batalha. As alterações afetam ações futuras e atualizam dinamicamente a timeline.

### Grid Tático

As batalhas acontecem sobre um grid lógico.

O grid é responsável por representar:

- posições;
- ocupação de tiles;
- validação de movimentação;
- terrenos transitáveis;
- estado do campo de batalha.

A representação visual é independente do estado lógico do grid.

### Câmera Isométrica

O jogo utiliza uma perspectiva isométrica 2D.

O jogador pode alternar a visualização do mapa entre quatro ângulos predefinidos.

A rotação da câmera não altera o estado lógico da batalha.

### Batalhas contra Bosses

Bosses são um dos principais elementos do jogo.

Um boss pode:

- ocupar múltiplas posições do grid;
- possuir partes ou regiões independentes;
- possuir vulnerabilidades específicas;
- modificar o campo de batalha;
- mudar de fase;
- alterar sua periculosidade durante o combate.

Os confrontos devem ser resolvidos através de decisões táticas e interação com as mecânicas da batalha, e não apenas reduzindo os pontos de vida do inimigo.

---

# MVP

O primeiro marco do projeto é focado em validar a experiência central de combate.

O MVP deve permitir que o jogador:

1. Inicie uma batalha.
2. Jogue em um mapa isométrico baseado em grid.
3. Alterne a câmera entre quatro ângulos predefinidos.
4. Controle unidades e as movimente pelo grid.
5. Acompanhe uma timeline dinâmica.
6. Perceba como velocidade e periculosidade afetam a frequência das ações.
7. Enfrente um boss que ocupa múltiplos tiles.
8. Identifique e ataque uma posição vulnerável.
9. Vença ou perca a batalha.
10. Reinicie o confronto.

## Checklist do MVP

### Projeto

- [ ] Projeto C++ configurado
- [ ] SDL3 integrada
- [ ] Contexto OpenGL configurado
- [ ] CMake configurado
- [ ] Build local funcionando

### Arquitetura

- [ ] Lógica do jogo separada das camadas de plataforma e renderização
- [ ] Módulo central de batalha definido
- [ ] Testes automatizados básicos configurados

### Grid

- [ ] Grid lógico
- [ ] Mapa de teste
- [ ] Conversão entre coordenadas do grid e tela
- [ ] Tiles transitáveis e bloqueados
- [ ] Sistema de ocupação de tiles

### Câmera

- [ ] Renderização isométrica
- [ ] Quatro ângulos predefinidos
- [ ] Rotação da câmera
- [ ] Estado lógico preservado durante a rotação

### Participantes

- [ ] Unidade controlada pelo jogador
- [ ] Inimigo básico
- [ ] Posição no grid
- [ ] Estado ativo/inativo
- [ ] Velocidade

### Timeline Dinâmica [Dynamic-Timeline.md](docs/Mechanics/Dynamic-Timeline.md)

- [ ] Sem rodadas fixas
- [ ] Velocidade influencia a frequência de ações
- [ ] Periculosidade influencia a frequência de ações
- [ ] Sistema determina o próximo participante
- [ ] Timeline futura pode ser exibida para depuração
- [ ] Alterações de velocidade afetam ações futuras
- [ ] Alterações de periculosidade afetam ações futuras

### Ações

- [ ] Início e fim de turno
- [ ] Validação de ações
- [ ] Movimentação
- [ ] Ataque básico
- [ ] Resolução de dano
- [ ] Tratamento de participantes derrotados

### Boss

- [ ] Boss ocupa múltiplos tiles
- [ ] Posição ou região vulnerável
- [ ] Pelo menos uma ação própria
- [ ] Atributo de periculosidade
- [ ] Alteração dinâmica de periculosidade

### Encerramento da Batalha

- [ ] Condição de vitória
- [ ] Condição de derrota
- [ ] Apresentação do resultado
- [ ] Reinício da batalha

---

# Princípios de Arquitetura

## Lógica do Jogo Independente da Plataforma

O núcleo do jogo não deve depender de:

- SDL3;
- OpenGL;
- renderização;
- câmera;
- animações;
- dispositivos de entrada;
- APIs específicas de navegador ou dispositivos mobile.

A arquitetura deve manter responsabilidades separadas:

```text
┌─────────────────────────────────┐
│       Plataforma / Entrada      │
│                                 │
│       SDL3 / Input / Áudio      │
└────────────────┬────────────────┘
                 │
                 │ Comandos
                 ▼
┌─────────────────────────────────┐
│         Núcleo do Jogo          │
│                                 │
│ Batalha                         │
│ Timeline                        │
│ Grid                            │
│ Participantes                   │
│ Ações                           │
└────────────────┬────────────────┘
                 │
                 │ Estado / Eventos
                 ▼
┌─────────────────────────────────┐
│         Renderização            │
│                                 │
│ OpenGL / Câmera / Sprites / UI  │
└─────────────────────────────────┘
```

O núcleo do jogo recebe comandos e atualiza o estado da batalha.

As camadas externas são responsáveis por:

- converter a entrada do jogador em comandos;
- apresentar o estado da batalha;
- reagir aos eventos produzidos pelo jogo.

## Lógica de Batalha Determinística

Dado o mesmo:

- estado inicial;
- mapa;
- conjunto de participantes;
- sequência de ações;
- sequência de valores aleatórios;

a batalha deve produzir o mesmo resultado.

Isso facilita:

- testes automatizados;
- reprodução de bugs;
- registro de batalhas;
- uma possível implementação futura de multiplayer.

## Composição em Vez de Herança

O projeto deve priorizar composição para definir mecânicas de combate.

Novas ações, efeitos, comportamentos de inimigos e mecânicas de bosses devem poder ser adicionados sem exigir alterações extensas no núcleo do sistema.

---

# Tecnologias

| Tecnologia | Finalidade |
|---|---|
| C++ | Linguagem principal |
| SDL3 | Janela, eventos, input e funcionalidades de plataforma |
| OpenGL | Renderização |
| CMake | Sistema de build |

## Plataformas

O projeto tem como objetivo suportar:

- Web;
- Dispositivos mobile.

O código específico de cada plataforma deve permanecer isolado da lógica central do jogo.

---

# Status do Projeto

O projeto está na fase de **design e planejamento do MVP**.

A prioridade atual é validar:

1. Timeline dinâmica.
2. Interação entre velocidade e periculosidade.
3. Movimentação tática.
4. Bosses que ocupam múltiplos tiles.
5. Encontros com bosses baseados em puzzle.

---

# Decisões Atuais

- O jogo será desenvolvido em C++.
- SDL3 será utilizada para funcionalidades de plataforma.
- OpenGL será utilizado para renderização.
- CMake será utilizado como sistema de build.
- As batalhas utilizarão uma timeline dinâmica em vez de rodadas fixas.
- Velocidade influencia a frequência das ações.
- Periculosidade aumenta a frequência efetiva de ações de participantes especiais.
- As batalhas acontecem sobre um grid lógico.
- A apresentação visual utiliza uma perspectiva isométrica 2D.
- A câmera possui quatro ângulos predefinidos.
- A rotação da câmera não altera o estado lógico da batalha.
- Bosses podem ocupar múltiplas posições do grid.
- Batalhas contra bosses possuem mecânicas baseadas em estratégia e puzzle.
- Multiplayer cooperativo não faz parte do escopo inicial do MVP.

---

# Próximos Passos

As próximas tarefas de design técnico são:

- ~~Definir o modelo matemático da timeline dinâmica.~~
- ~~Definir o modelo de domínio do sistema de batalha.~~
- Definir a estrutura inicial do projeto.
- Criar o primeiro protótipo jogável de batalha.