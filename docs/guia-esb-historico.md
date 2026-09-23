# roBa KESB: guia rápido

Keymap salvo em 16/09/2026 pelo ZMK Studio, com 10 layers.

## Como ler este guia

- **Posições:** cada tecla é chamada pela letra que ela tem na layer **Mac**. Por exemplo, "Numbers + W" é a tecla do W com a layer Numbers ativa.
- **Polegares (da esquerda para a direita):**
  - **P1:** polegar externo esquerdo
  - **P2:** polegar do meio esquerdo
  - **P3:** polegar interno esquerdo
  - **P4:** polegar interno direito
  - **P5:** polegar externo direito
- **Teclas internas:** são as colunas extras perto do centro. "Interna-G" fica ao lado do G, "Interna-H" ao lado do H, "Interna-B" ao lado do B e "Interna-N" ao lado do N.
- **Cluster:** os 5 botões à direita, B1 a B5 da esquerda para a direita.
- **Teclas transparentes (▽):** repassam o que estiver na Mac.

## Layers e como chegar em cada uma

| Layer | Como ativar | Tipo |
|---|---|---|
| **Mac** | Base. Na System, a tecla M volta pra Mac | Base |
| **Numbers** | Segurar **P2** | Momentânea |
| **Symbols** | Segurar **P5** | Momentânea |
| **System** | Segurar **P2 + P5** juntos (Numbers + Symbols). É conditional layer, não tem tecla própria | Conditional |
| **Mouse** | Segurar **P1**. A tecla abaixo do `/` (`mo_tog`) também ativa: segurar = momentânea, **tocar (< 200 ms) = TRAVA ligada**. Também há um combo: `,` + `.` + `/` + a tecla abaixo do Z | Momentânea / trava |
| **Tile Mac** | Segurar **A** ou **;** (tocar digita a letra) | Layer-Tap |
| **Scroll** | Numbers: segurar **X**. Mouse: tecla **X** ou **/** (`mo_tog`) | Momentânea / trava |
| **Snipe** | Numbers: segurar **Z**. Mouse: segurar **Z** ou **/** | Momentânea |
| **Fast Scroll** | Segurar o botão **MB4** do mouse Lariska | Momentânea |
| **H-Scroll** | **MB4 + MB5** juntos (Fast Scroll + Tiling). É conditional layer | Conditional |

Scroll, Snipe, Fast Scroll e H-Scroll não mudam nenhuma tecla. O efeito está no trackball, configurado no firmware (`roBakesb.overlay`) e invisível no ZMK Studio:

- **Scroll:** o movimento da bola vira rolagem, com eixo Y invertido e velocidade dividida por 10.
- **Snipe:** movimento do cursor dividido por 3, ou seja, modo de precisão.

**A layer Mouse NÃO liga sozinha ao mexer na trackball.** Não existe `zip_temp_layer` no config. Se o teclado parecer "preso" no modo mouse, foi a tecla `mo_tog` tocada rápido ou o combo.

## Teclas comuns

| Quero… | Faça |
|---|---|
| **Espaço** | Tocar **P3** |
| **Enter** | Tocar **P4** |
| **Shift** | Segurar **P3** ou **P4** |
| **Backspace** | P2 + **;** |
| **Delete** | P2 + **/** |
| **Tab** | P2 + **D** |
| **Esc** | P2 + **F** |
| **Setas ← ↓ ↑ →** | P2 + **H J K L** |
| **Home / PgDn / PgUp / End** | P2 + **N / M / , / .** |
| **Ctrl** | Segurar **S** (esq.), **L** ou **/** (dir.) |
| **Alt/Option** | Segurar **D** (esq.), **K** ou **.** (dir.) |
| **Cmd** | Segurar **F** (esq.) ou **J** (dir.) |
| **Números 1–0** | P2 + **Q W E R T Y U I O P** |
| **F1–F10** | System + **Q … P** |
| **F11 / F12** | System + **A / S** |
| **Print (Cmd+Shift+4)** | **Interna-G** na Mac (ou System + F) |
| **Cmd+Shift+3** | System + **D** |
| **Cmd+Espaço (Spotlight)** | Segurar A (Tile Mac) + tocar **P3** |
| **Alt+Espaço** | **Interna-B** na Mac |
| **Ctrl+↑ (Mission Control)** | Tecla **abaixo do C** na Mac |
| **Cliques do mouse** | Cluster **B1 / B2 / B3** = botão 1 / 2 / 3. Na Numbers e na Mouse, os botões 2 / 1 / 3 também ficam em **C / V / B** |

## Símbolos (segurar P5)

Pressupõe layout US no computador.

| Tecla | Q | W | E | R | T | Y | U | I | O | P |
|---|---|---|---|---|---|---|---|---|---|---|
| Símbolo | ! | @ | # | $ | % | ^ | & | * | \| | \\ |

| Tecla | A | S | D | F | G | H | J | K | L | ; |
|---|---|---|---|---|---|---|---|---|---|---|
| Símbolo | { | } | + | = | - | _ | ( | ) | " | : |

| Tecla | Z | X | C | V | M | , | . | / |
|---|---|---|---|---|---|---|---|---|
| Símbolo | ` | ~ | [ | ] | ' | < | > | ? |

## System (mídia, conexão)

| Tecla | Ação |
|---|---|
| H / J / K | Faixa anterior / Play-Pause / Próxima |
| L / ; / / | Volume − / Volume + / Mudo |
| G | PrintScreen |
| Interna-G e Interna-B | Saída USB |
| Interna-H e Interna-N | Saída BLE (sem efeito prático: este firmware é USB via dongle, `CONFIG_ZMK_BLE=n`) |
| Z X C V B | `bt_macro_0` a `bt_macro_4`: trocam a layer base e selecionam o perfil Bluetooth 0 a 4. ⚠️ As de V e B (`bt_macro_3`/`4`) fazem `&to Windows`, layer que foi removida |
| M | Voltar para Mac |
| N e , | Sem função (antes iam pra Gaming e Windows, que foram removidas) |

## Tile Mac (segurar A ou ;)

| Tecla | Toque | Segurar |
|---|---|---|
| Q … O (1ª linha, 9 teclas) | Shift+Alt+1…9 | Ctrl+Shift+Alt+1…9 |
| D / F / G | Shift+Alt+E / R / T | Ctrl+Shift+Alt+E / R / T |
| S | Ctrl+F4 | — |
| Z / C | `SWAPPERMAC_REV` / `SWAPPERMAC` (provavelmente alternam janelas, pra trás e pra frente) | — |
| X | Ctrl+Shift+F4 | — |
| P3 | Cmd+Espaço | — |

Pelas combinações, deve ser pra um gerenciador de janelas/tiling. Não confirmei qual app usa esses atalhos.

## Behaviors customizados sem descrição no Studio

`deskhop_gaming` (Interna-H), `deskhop_normal` (Interna-N), `flush_mods` (abaixo do Z), `td_mb4_mac` (B4) e `MO_MKP` (B5) vêm do firmware. O Studio só mostra o nome, então o que cada um faz exatamente precisa ser conferido no `.keymap`.

## Trocar o teclado de computador (Bluetooth)

Vale para o firmware novo, com a metade direita como central. O ZMK tem 5 perfis Bluetooth e conecta a um host por vez. Pela [doc do ZMK](https://zmk.dev/docs/features/bluetooth), para parear com um novo computador é preciso "selecionar um perfil não usado, ou limpar o perfil atual": um perfil já pareado **não** é substituído sozinho quando outro computador tenta parear. É por isso que o teclado aparece na lista do computador novo, mas o Conectar não completa.

### Mudar para um computador novo (usando um computador por vez)

1. **No computador antigo:** esquecer o "roBa" nas configurações de Bluetooth. Se ele não estiver por perto, faça isso depois, senão ele vai tentar reconectar sempre que você estiver perto dele.
2. **No teclado:** segure **P2 + P5** (layer System) e toque **Z**, para garantir que o perfil 0 é o ativo.
3. **Ainda com P2 + P5 segurados**, toque **`.`** (`BT_CLR`). Isso apaga o pareamento **do perfil ativo** e o teclado volta a ficar visível.
4. **No computador novo:** procurar o "roBa" no Bluetooth e conectar.

Se o computador novo for **Windows**, use **V** no passo 2 em vez de Z: essa tecla seleciona o perfil 3 e já muda a layer base para Win.

### Usar dois computadores ao mesmo tempo

Um perfil para cada, sem apagar nada:

1. No computador novo, esquecer o "roBa" se ele já estiver listado.
2. No teclado: **P2 + P5** e toque **X** (perfil 1).
3. Opcional: toque **`.`** para garantir que o perfil 1 está limpo.
4. Parear pelo computador novo.

Para alternar depois, é só a layer System: **Z** = perfil 0, **X** = 1, **C** = 2 (os três também trocam a base para Mac), **V** = 3 e **B** = 4 (esses dois trocam a base para Win).

## Problemas conhecidos e contornos

| Sintoma | Causa | O que fazer |
|---|---|---|
| Tecla não sai, fica "pressionada" ou layer presa, com ou sem trackball | Perda de pacote no link ESB (2,4 GHz): após 8 retransmissões o ESB desiste, e a cura por keepalive do módulo não está funcionando | Aproximar o dongle; como solução, migrar para Bluetooth (`roba-migracao-bluetooth.md`). `CONFIG_ESB_TX_FIFO_SIZE=16` só ajuda se houver rajadas |
| Layer fica travada (ex.: D digitando `+` = Symbols presa) | O pacote de "soltar" o polegar se perdeu; o `&mo` é por evento, não relê o estado da tecla | Apertar e soltar o mesmo polegar de novo. Se não resolver, reconectar o dongle |
| Teclas saem deslocadas uma posição (W dá Q, D dá S) | Physical layout errado após restore de settings | Reconectar o dongle. Se persistir, conferir o campo "Layout:" no Studio |
| Cursor demora pra responder após uma pausa | O sensor PMW3610 entra em modo de descanso | Normal. Ajustável nos `CONFIG_PMW3610_REST*` |

## Atualizar o módulo ESB (e ajustar a fila de TX)

O link entre as metades e o dongle usa **ESB**, protocolo da Nordic, pelo módulo [damex/zmk-feature-split-esb](https://github.com/damex/zmk-feature-split-esb). O config atual está na `v0.6.1`. Em 22/09/2026 as tags mais novas são `v0.6.2`, `v0.6.3` e `v0.6.4` ([tags](https://github.com/damex/zmk-feature-split-esb/tags)). A `v0.6.4` (21/09) passou a aplicar os patches do nRF automaticamente; o workflow deste repositório ainda roda `west patch` por conta própria, então teste a compilação antes de gravar.

**O ajuste mais importante:** o README do módulo diz que em periférico com fonte de eventos em rajada a fila padrão "drops events at the source under a burst", e recomenda `CONFIG_ESB_TX_FIFO_SIZE=16`. A variante `roBaesb` do repositório define isso; o **`roBakesb` nunca definiu** (`git log -S ESB_TX_FIFO_SIZE` na pasta do shield não retorna nada). O commit `1245702` ("bump esb to v0.6.1") removeu outra propriedade, `CONFIG_ZMK_SPLIT_ESB_RX_THREAD_STACK_SIZE`, que deixou de existir na v0.6.x. `CONFIG_ESB_TX_FIFO_SIZE` é um símbolo do nRF Connect SDK, não do módulo, e continua válido.

### Passos

1. Fazer **fork** de `charlesmst/zmk-config-generic`.
2. Em `config/west.yml`, trocar a revisão do `zmk-feature-split-esb` de `v0.6.1` para a versão desejada.
3. Adicionar `CONFIG_ESB_TX_FIFO_SIZE=16` em:
   - `config/boards/shields/roBakesb/roBakesb_left.conf`
   - `config/boards/shields/roBakesb/roBakesb_right.conf`
4. Commit e push. O workflow `.github/workflows/build.yml` compila tudo que está em `build.yaml`.
5. Baixar os artifacts da execução: `roBakesb_left`, `roBakesb_right` e o do dongle.
6. **Gravar nos três dispositivos.** As duas pontas do protocolo precisam estar na mesma versão.

### Gravação

**Metades (XIAO nRF52840):** duplo toque no botão de reset. Aparece um drive USB. Copiar o `.uf2` para dentro dele.

**Dongle nice!nano:** mesmo procedimento UF2.

**Dongle Holyiot:** não é UF2. Pelo README do repositório:

```powershell
nrfutil pkg generate --hw-version 52 --sd-req 0x00 --application .\roBakesb_dongle_holyiot.hex --application-version 1 .\roBakesb_dongle_holyiot.zip
nrfutil device list --traits nordicDfu
nrfutil dfu usb-serial -pkg .\roBakesb_dongle_holyiot.zip -p COM16 -b 115200 -t 60
```

Para entrar em modo DFU: aproximar um ímã da área de reset da placa até o LED vermelho piscar em respiração. A porta COM muda entre o modo normal e o DFU, então rode o `device list` já em DFU.

⚠️ **Não faça "Restore Stock Settings" sem necessidade.** As mudanças feitas pelo ZMK Studio (layers removidas, teclas em None) ficam na memória de configuração do dongle, não no firmware. Regravar o firmware normalmente preserva; o restore apaga e volta às 15 layers do `.keymap`.

⚠️ Guarde os `.uf2` atuais antes de atualizar, para conseguir voltar.

## Remapear no `.keymap`

Arquivo: `config/roBakesb.keymap` no repositório. É o que corresponde ao "roBa KESB". Os outros (`roBa.keymap`, `roBaesb.keymap`) são de outras montagens.

### Estrutura

Cada layer tem **48 bindings**, na ordem das posições 0 a 47:

| Posições | Onde ficam |
|---|---|
| 0–4 / 5–9 | 1ª linha: esquerda (Q W E R T) / direita (Y U I O P) |
| 10–15 / 16–21 | 2ª linha: A S D F G + interna-G / interna-H + H J K L `;` |
| 22–27 / 28–33 | 3ª linha: Z X C V B + interna-B / interna-N + N M `,` `.` `/` |
| 34–36 | 4ª linha esquerda (abaixo de Z, X, C) |
| 37–41 | Polegares P1 a P5 |
| 42 | Tecla abaixo da `/` |
| 43–47 | Botões do mouse Lariska |

As layers têm apelidos definidos no topo do arquivo: `DEFAU 0`, `WINDO 1`, `GAMIN 2`, `NUMBE 3`, `SYMBO 4`, `LOGAM 5`, `SUPER 6`, `MOUSE 7`, `TILIN 8`, `TILWI 9`, `SCROL 10`, `SNIPE 11`, `GAMFN 12`, `FSCRL 13`, `HSCRL 14`.

### Behaviors mais usados

| Escrita | O que faz |
|---|---|
| `&kp A` | Digita A |
| `&kp LG(LS(N4))` | Cmd+Shift+4 |
| `&mo NUMBE` | Layer momentânea enquanto segura |
| `&lt TILIN A` | Segurar = layer Tile Mac, tocar = A |
| `&hm LCTRL S` | Segurar = Ctrl, tocar = S (homerow mod) |
| `&tog GAMIN` | Liga/desliga a layer |
| `&to DEFAU` | Troca a layer base |
| `&trans` | Transparente: repassa para a layer de baixo |
| `&none` | Não faz nada |
| `&bd LCLK` | Clique do mouse |

### Exemplo de mudança

Para tirar a trava acidental do modo mouse, na layer Mac, posição 42:

```dts
/* antes */   MO_TOG(MOUSE)
/* depois */  &mo MOUSE
```

Assim a tecla só ativa enquanto segurada e nunca trava.

### Regras que só existem no `.keymap`

Coisas que o ZMK Studio **não mostra nem edita**:

- **Conditional layers:** `NUMBE + SYMBO → SUPER` (System), `FSCRL + TILIN → HSCRL`.
- **Combos:** `,` `.` `/` + a tecla abaixo do Z alternam a layer Mouse. Na Windows, X+C+V alternam a Gaming.
- **Macros:** `deskhop*`, `bt_macro_*`, `flush_mods`, `dmouse`.
- **Input processors do trackball:** as escalas de Scroll e Snipe.

Depois de editar, o ciclo é o mesmo da seção anterior: commit, o Actions compila, você grava nos três dispositivos.

---
Backup do estado anterior (15 layers): `roba-kesb-keymap-backup-2026-09-16.json` / `.html`
