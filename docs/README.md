# roBa — configuração do Erick (Bluetooth)

Documentação do meu teclado roBa. Este fork vem de [charlesmst/zmk-config-generic](https://github.com/charlesmst/zmk-config-generic).

**A minha configuração vive na branch [`bt-zmk-0.3`](../../tree/bt-zmk-0.3), não na `main`.** A `main` segue o repositório de origem, com ESB e dongle; a `bt-zmk-0.3` é a versão Bluetooth, com a metade direita como central e sem dongle.

## Explorador do keymap

**[Abrir o explorador interativo](https://erickmobltda.github.io/zmk-config-generic/)**

Mostra todas as teclas em todas as layers, com a comparação entre o firmware antigo (ESB, 12 layers) e o atual (7 layers). Clique numa tecla para ver o que ela faz em cada layer.

## Como está montado hoje

| | |
|---|---|
| Transporte entre as metades | Bluetooth (BLE) |
| Central | Metade **direita** (a do trackball) |
| Conexão com o computador | Bluetooth, ou cabo USB na metade direita |
| Dongle | Não é usado |
| Firmware | Fork do ZMK 0.3, branch `bt-zmk-0.3` |
| Placas | Seeed XIAO nRF52840 nas duas metades |

### Builds

O `build.yaml` desta branch compila três alvos:

| Arquivo | Vai para |
|---|---|
| `roBa_R-seeeduino_xiao_ble-zmk.uf2` | Metade **direita** (central, com ZMK Studio) |
| `roBa_L_enc_inv.uf2` | Metade **esquerda** |
| `settings_reset-seeeduino_xiao_ble-zmk.uf2` | Só em emergência, quando o pareamento trava |

### Gravar

Duplo clique no reset da metade → aparece um drive USB → copiar o `.uf2` para a raiz. A placa reinicia sozinha.

## Layers

| # | Layer | Como ativar |
|---|---|---|
| 0 | Mac | Base |
| 1 | Number | Segurar **P2** (2º polegar da esquerda) |
| 2 | Symbol | Segurar **P5** (polegar externo direito) |
| 3 | Super | **P2 + P5** juntos (conditional layer) |
| 4 | Mouse | Segurar **P1** (polegar externo esquerdo) |
| 5 | Scroll | Number + **X**, ou pela Mouse. Faz a trackball rolar a tela |
| 6 | Snipe | Number + **Z**, ou pela Mouse. Cursor 3× mais lento |

Scroll e Snipe não têm teclas: são as layers que os *input processors* do trackball observam, definidas em `config/boards/shields/roBa/roBa.dtsi`. Se renumerar as layers, **é preciso atualizar os `layers = <5>` e `<6>` nesse arquivo**.

## Bluetooth

Três perfis, na layer **Super** (P2 + P5):

| Tecla | Ação |
|---|---|
| **Z** | Perfil 0 |
| **X** | Perfil 1 |
| **C** | Perfil 2 |
| **.** | `BT_CLR` — apaga o pareamento do perfil ativo |

**Parear um computador novo:** selecionar um perfil livre → se o computador não enxergar o teclado, dar `BT_CLR` nesse perfil → parear pelo computador.

**Se um computador esqueceu o teclado:** é preciso limpar o perfil correspondente com `BT_CLR` também no teclado, senão ele fica tentando reconectar em vez de se anunciar, e nenhum computador o encontra.

## Documentos

- [Changelog desta branch](CHANGELOG.md) — o que mudou, quando e por quê
- [Guia de migração do dongle ESB para Bluetooth](migracao-bluetooth.md) — o passo a passo completo, com backup e caminho de volta
- [Guia do firmware antigo, com ESB e dongle](guia-esb-historico.md) — histórico, para consulta se eu voltar ao dongle

## Sincronizar com o repositório de origem

```bash
git fetch upstream
git checkout bt-zmk-0.3
git merge upstream/main     # ou rebase, se preferir
```

Esta pasta `docs/` só existe nesta branch, então ela não gera conflito com o repositório de origem.
