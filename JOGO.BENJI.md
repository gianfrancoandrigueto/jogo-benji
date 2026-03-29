<role>
Você é um engenheiro de jogos AAA especializado em fighting games competitivos (nível torneio), com experiência em:

- Frame data (startup, active, recovery)
- Sistemas de treino estilo Street Fighter / Tekken
- Debug tools para jogos de luta
- Input buffer e leitura de comandos avançados
- Arquitetura de engine para jogos competitivos

Seu objetivo NÃO é apenas criar um jogo, mas sim uma ferramenta de TREINO PROFISSIONAL.
</role>

<objective>
Evoluir o jogo de luta existente para incluir um MODO PRO PLAYER com ferramentas avançadas de treino, análise e debug.

Tudo deve rodar em um único arquivo HTML, sem bibliotecas externas.
</objective>

<constraints>
- Código completo e funcional
- Nenhum pseudo-código
- Alta precisão de timing (baseado em frames)
- Interface clara e utilizável
</constraints>

---

<training_mode>
Implementar um modo de treino completo com:

- Vida infinita (opcional)
- Energia infinita (opcional)
- Reset rápido de posição
- Dummy configurável (parado, defender, contra-atacar)

Configurações do dummy:
- Defender sempre
- Defender após primeiro hit
- Contra-atacar após block
</training_mode>

---

<frame_data_system>
Sistema de frame data em tempo real:

Exibir na tela:

- Startup frames do golpe atual
- Active frames
- Recovery frames
- Frame advantage (+ ou - após hit/block)

Atualizar em TEMPO REAL conforme ações do jogador.

IMPORTANTE:
Sistema deve calcular vantagem baseado na interação real entre os personagens.
</frame_data_system>

---

<input_display>
Mostrar na tela:

- Últimos inputs do jogador
- Sequência de comandos (ex: → ↓ ↘ + ataque)
- Buffer de inputs

Formato estilo jogos profissionais (linha horizontal de inputs).
</input_display>

---

<hitbox_viewer>
Modo debug ativável com tecla:

Exibir visualmente:

- Hitboxes (vermelho)
- Hurtboxes (verde)
- Pushboxes (azul)

Atualização frame a frame.

Permitir ligar/desligar durante a luta.
</hitbox_viewer>

---

<combo_system>
Sistema avançado de combos:

- Contador de hits
- Dano total do combo
- Reset automático ao cair combo

Exibir:
- "Combo X hits"
- "Damage: XXX"

Detectar combos reais (não apenas spam de ataque).
</combo_system>

---

<replay_system>
Sistema de replay simples:

- Gravar inputs do jogador
- Reproduzir sequência

Permitir:
- Treinar contra seu próprio padrão
</replay_system>

---

<frame_step_mode>
Modo frame-by-frame:

- Pausar o jogo
- Avançar 1 frame por vez

Essencial para análise técnica.

Ativação por tecla.
</frame_step_mode>

---

<ai_training>
IA avançada para treino:

- Repetir ações específicas
- Simular combos
- Punir automaticamente erros do jogador

Exemplo:
- Se jogador errar golpe → IA contra-ataca
</ai_training>

---

<ui_debug>
Painel de debug com:

- FPS
- Estado atual do personagem
- Frame atual da animação
- Estado da IA

Visual discreto, estilo dev tool.
</ui_debug>

---

<architecture_requirements>
Separar claramente:

- TrainingSystem
- DebugSystem
- FrameDataSystem
- ReplaySystem

Código modular e escalável.
</architecture_requirements>

---

<controls>
Definir teclas para:

- Ativar/desativar hitbox viewer
- Ativar modo treino
- Pausar jogo
- Frame step
- Reset posição
</controls>

---

<output_format>
Entregar:

1. Código completo (HTML único)
2. Comentado como ferramenta profissional
3. Indicação clara de:
   - onde alterar frame data
   - onde configurar golpes
   - onde ajustar IA
</output_format>

---

<final_validation>
Validar:

- Frame data correto
- Hitboxes funcionando
- Input display preciso
- Combo counter confiável
- Frame step funcional
- Replay funcionando

Se algo falhar, corrigir antes de entregar.
</final_validation>