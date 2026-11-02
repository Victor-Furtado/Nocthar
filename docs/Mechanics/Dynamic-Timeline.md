## Timeline Dinâmica

O combate utiliza uma timeline dinâmica, na qual cada participante recebe oportunidades de ação de acordo com sua frequência efetiva. Não existem rodadas fixas.

### Frequência Efetiva

A frequência de ações é determinada pela combinação de dois atributos:

F = V × D

- **Velocidade (V):** determina a frequência básica de ações do participante.
- **Periculosidade (D):** multiplica a frequência de participantes especiais, como bosses.

Para limitar a diferença entre participantes rápidos e lentos, a frequência efetiva passa por uma curva de redução baseada na raiz quadrada:

Fₐ = √F

Onde:
- **F:** frequência original.
- **Fₐ:** frequência ajustada.

O intervalo necessário para realizar uma ação é:

Δt = 1 / Fₐ

Ou, de forma expandida:

Δt = 1 / √(V × D)

### Progresso de Ação

Cada participante possui um progresso individual entre 0 e 1, representando o avanço em direção à próxima ação.

O progresso aumenta continuamente de acordo com sua frequência ajustada:

dP/dt = Fₐ

Quando o progresso alcança 1, o participante fica apto a agir. Após executar sua ação, seu progresso é reduzido em 1.

### Determinação do Próximo Participante

A timeline calcula quanto tempo falta para cada participante completar seu progresso:

Δtᵢ = (1 - Pᵢ) / Fₐᵢ

O menor intervalo determina o próximo instante de ação.

Todos os participantes avançam seu progresso simultaneamente durante esse intervalo. Caso múltiplos participantes alcancem progresso 1 no mesmo instante, a ordem de execução é determinada por prioridade.

### Alterações Durante a Batalha

Alterações de velocidade ou periculosidade modificam imediatamente a frequência ajustada, sem reiniciar o progresso acumulado.

Isso permite que habilidades, efeitos e mudanças de fase dos bosses influenciem dinamicamente o ritmo do combate.

### Regras Gerais

- Não existem rodadas fixas.
- Todos os participantes começam com progresso zero.
- O progresso é preservado entre ações.
- Participantes derrotados ou inativos não acumulam progresso.
- Frequência zero impede a acumulação de progresso.
- Eventos simultâneos são resolvidos por prioridade.
- Empates de prioridade são resolvidos por ID, garantindo determinismo.

**Objetivo:** criar um sistema de combate contínuo no qual velocidade, periculosidade e efeitos de batalha determinam dinamicamente a frequência das ações, mantendo diferenças controladas entre participantes rápidos e lentos.