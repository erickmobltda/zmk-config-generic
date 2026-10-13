# roBa: migrar do dongle ESB para Bluetooth (metade direita como central)

Guia para quem nunca fez isso. Leia tudo uma vez antes de começar.

## Visão geral

**Hoje:** metade esquerda e metade direita → ESB (2,4 GHz) → dongle USB → Mac.

**Depois:** metade esquerda → Bluetooth → **metade direita (central)** → Bluetooth → Mac. Sem dongle.

**Por que isso deve ajudar:** os sintomas (tecla que não sai, tecla ou layer que fica "presa") acontecem mesmo sem mexer na trackball, então apontam para o transporte ESB em si, não para rajadas do sensor. Os dois protocolos tratam falha de entrega de forma diferente:

- **ESB (hoje):** pela documentação da Nordic, o transmissor reenvia até o número máximo configurado (`retransmit-count = 8` no seu config) e então desiste com `ESB_EVENT_TX_FAILED`. Depois disso, a recuperação depende do módulo `zmk-feature-split-esb`, que é de terceiros e está em versão 0.x. O próprio documento de handoff do repositório diz que o problema de tecla presa "ainda não está comprovadamente resolvido de ponta a ponta".
- **BLE (depois):** pelo material da Bluetooth SIG, o link layer reenvia o pacote com o mesmo número de sequência até receber a confirmação. Sem limite de tentativas: ou o pacote chega, em ordem, ou a conexão cai por timeout. Além disso, o transporte BLE de split é o nativo do ZMK.

⚠️ Isso não prova que o BLE vai eliminar os seus sintomas: pode haver bug em outro ponto da pilha (o fork do ZMK é customizado). É a hipótese mais provável e o teste de menor custo.

**O que muda para você:**

| | Hoje | Depois |
|---|---|---|
| Conexão com o Mac | Dongle USB | Bluetooth (ou cabo USB na metade direita) |
| Bateria | Parecida nas duas metades | **A direita gasta mais**; pela doc do ZMK, o central gasta "significativamente mais" |
| ZMK Studio | Via dongle | Cabo USB na **metade direita** |
| Latência entre metades | ESB | BLE: +3,75 ms em média, 7,5 ms no pior caso (doc do ZMK). Imperceptível para digitar |
| Versão do ZMK | Fork novo (branch `main`) | Fork ZMK 0.3 (branch `bt-zmk-0.3`) |
| Keymap | 15 layers (`roBakesb.keymap`) | 11 layers (`roBa.keymap`). Veja a seção 9 |

**O que você precisa:**

- Mac com Terminal e `git` (já vem com as Command Line Tools)
- Conta no GitHub com o fork `erickmobltda/zmk-config-generic`
- Cabo USB-C **de dados** (não só de carga)
- As duas metades carregadas
- Uns 60–90 minutos na primeira vez

**Tempo de "teclado fora do ar":** do passo 5 ao 6, uns 10 minutos. Tenha outro teclado por perto.

---

## 1. Backup e plano de volta (faça ANTES de mudar qualquer coisa)

O dongle **não vai ser regravado**; ele só fica guardado. Para voltar ao setup atual, basta restaurar o firmware das duas metades. Há dois caminhos de volta; faça o backup dos dois.

### 1.1 Identifique as metades

Cole uma fita com "E" e "D" em cada metade. Os arquivos de firmware são diferentes para cada lado e gravar trocado não funciona.

### 1.2 Copie o firmware atual de cada metade (CURRENT.UF2)

O bootloader UF2 da Adafruit, usado em placas nRF52840 como o XIAO, expõe no drive USB um arquivo `CURRENT.UF2` com o conteúdo atual da memória. Confirmei que esse arquivo existe no código-fonte do bootloader (`src/usb/uf2/ghostfat.c`). **Não testei restaurar a partir dele**, por isso a 1.3 é o plano principal e esta é a rede de segurança extra.

Para cada metade, uma de cada vez:

1. Ligue a metade no Mac com o cabo USB-C.
2. **Duplo clique rápido no botão de reset.** É o botão minúsculo do XIAO, ao lado do conector USB-C, ou um botão de reset da placa, se o seu case expõe um. A doc do bootloader diz que são dois resets em até 500 ms.
3. Aparece um drive USB novo no Finder. Nesse momento a metade não funciona como teclado; é esperado.
4. Abra o `INFO_UF2.TXT` do drive e confira se menciona XIAO / nRF52840.
5. Se existir `CURRENT.UF2`, copie para uma pasta segura, renomeando:
   - metade esquerda → `backup-esquerda-CURRENT.UF2`
   - metade direita → `backup-direita-CURRENT.UF2`
6. Aperte reset **uma vez** (ou desconecte o cabo) para sair do bootloader. A metade volta a funcionar como antes.

Se não existir `CURRENT.UF2`, siga só com a 1.3.

### 1.3 Garanta que dá para recompilar o firmware atual

O firmware ESB das metades vem da branch `main` do repositório. No seu fork, a `main` está hoje em `0b1d5a9` ("bump esb to v0.6.2"). **Mas as metades muito provavelmente estão com uma build da era v0.6.1**, porque o dongle também está, e as duas pontas precisam falar o mesmo protocolo.

O último commit com ESB v0.6.1 é o **`1afae2e`** ("deskhop: clear the gaming layer on an output switch"). É o ponto de volta mais seguro. ⚠️ Não tenho como confirmar de qual commit exato veio o firmware gravado hoje.

Crie uma tag para não perder a referência. No Terminal:

```bash
git clone https://github.com/erickmobltda/zmk-config-generic.git
cd zmk-config-generic
git tag backup-esb-v0.6.1 1afae2e
git push origin backup-esb-v0.6.1
```

### 1.4 Guarde o dongle

Desconecte e guarde junto com o cabo. Ele não será usado na opção A, e é a peça mais chata de regravar (o Holyiot exige `nrfutil` e ímã).

---

## 2. Preparar o repositório

A branch `bt-zmk-0.3` existe no repositório do Charles, mas **não está no seu fork**: quando você fez o fork, só a `main` veio junto.

### 2.1 Trazer a branch para o seu fork

No Terminal, dentro da pasta clonada na 1.3:

```bash
git remote add upstream https://github.com/charlesmst/zmk-config-generic.git
git fetch upstream bt-zmk-0.3
git checkout -b bt-zmk-0.3 upstream/bt-zmk-0.3
git push -u origin bt-zmk-0.3
```

Confira no GitHub: o seletor de branches do seu fork agora deve mostrar `bt-zmk-0.3`.

### 2.2 Habilitar o GitHub Actions no fork

O GitHub desliga os workflows em forks por padrão.

1. Abra `https://github.com/erickmobltda/zmk-config-generic/actions`.
2. Se aparecer um aviso dizendo que os workflows não rodam neste fork, clique no botão verde para habilitá-los.

---

## 3. Ajustes antes de compilar (recomendado)

### 3.1 Compilar só o que você precisa

O `build.yaml` dessa branch compila umas 40 combinações (Charybdis, dongles, variantes ESB etc.). Isso demora e, se qualquer uma falhar, o zip único `firmware` não é gerado. Deixe só as três que você vai usar.

Substitua **todo** o conteúdo de `build.yaml` (na raiz do repositório, branch `bt-zmk-0.3`) por:

```yaml
---
include:
  - board: seeeduino_xiao_ble
    shield: roBa_R
    snippet: studio-rpc-usb-uart
  - board: seeeduino_xiao_ble
    shield: roBa_L_enc_inv
    artifact-name: roBa_L_enc_inv
  - board: seeeduino_xiao_ble
    shield: settings_reset
```

**Por que `roBa_L_enc_inv` e não `roBa_L`:** as duas habilitam o encoder da esquerda; a diferença é a ordem dos pinos A/B, ou seja, o sentido da rotação. O seu firmware atual (`roBakesb_left.overlay`) usa `a-gpios = D0` e `b-gpios = D5`, igual ao `roBa_L_enc_inv`. Assim o encoder gira no mesmo sentido de hoje. Se você não tiver encoder, não faz diferença.

### 3.2 Remover duas armadilhas do keymap

O `config/roBa.keymap` dessa branch tem duas diferenças perigosas em relação ao que você usa hoje:

- A tecla **abaixo do Z** é `&sys_reset`: **reinicia o teclado** ao ser apertada. Hoje ela é `flush_mods`.
- O polegar **P1** é `MO_TOG(MOUSE)`: segurar ativa a Mouse, mas **um toque rápido a trava ligada**. Hoje o P1 é `&mo MOUSE`, só momentâneo.

Nas layers **Mac** (linha 298) e **Win** (linha 310), troque o início da 4ª linha:

```dts
/* antes */
&sys_reset           &trans               &kp LG(LC(SPACE))    MO_TOG(MOUSE)        &mo NUMBE ...

/* depois */
&none                &trans               &kp LG(LC(SPACE))    &mo MOUSE            &mo NUMBE ...
```

Só mude esses dois bindings e mantenha o resto da linha igual. A linha 321 (Gaming) também tem `&sys_reset` na mesma posição; troque por `&none` se quiser.

### 3.3 Commit e push

```bash
git add build.yaml config/roBa.keymap
git commit -m "roBa BLE: build so R/L_enc_inv/settings_reset; remove sys_reset e trava do P1"
git push
```

---

## 4. Compilar e baixar

1. O push dispara a compilação. Acompanhe em **Actions** no seu fork. Se não disparar, abra o workflow **Build**, clique em **Run workflow** e escolha a branch `bt-zmk-0.3`.
2. Espere todos os jobs ficarem verdes (alguns minutos).
3. Na página da execução, role até **Artifacts** e baixe **`firmware`** (um `.zip`).
4. Descompacte. Devem existir:

| Arquivo | Vai para |
|---|---|
| `roBa_R-seeeduino_xiao_ble-zmk.uf2` | Metade **direita** |
| `roBa_L_enc_inv.uf2` | Metade **esquerda** |
| `settings_reset-seeeduino_xiao_ble-zmk.uf2` | **As duas**, antes do firmware |

Os nomes seguem a regra do workflow oficial do ZMK v0.3.0: sem `artifact-name`, o arquivo vira `<shield>-<board>-zmk`.

**Se um job falhar:** abra o job, veja o erro e me mande. Não grave nada.

---

## 5. Gravar

Siga a ordem **exatamente**. É o procedimento oficial de settings reset do ZMK para split: primeiro o reset nas duas partes, depois o firmware em cada uma.

### 5.1 Settings reset na esquerda

1. Cabo na metade **esquerda**, duplo clique no reset. O drive aparece.
2. Arraste `settings_reset-seeeduino_xiao_ble-zmk.uf2` para a raiz do drive.
3. O drive some sozinho; a placa reinicia. Se o macOS reclamar que o disco não foi ejetado corretamente, pode ignorar: a placa desmonta sozinha ao reiniciar.

### 5.2 Settings reset na direita

Mesmo procedimento, com o mesmo arquivo, na metade **direita**.

### 5.3 Firmware na esquerda

Duplo reset na **esquerda** → arraste `roBa_L_enc_inv.uf2`.

### 5.4 Firmware na direita

Duplo reset na **direita** → arraste `roBa_R-seeeduino_xiao_ble-zmk.uf2`.

### 5.5 Deixe as metades se acharem

Desconecte o cabo, deixe as duas ligadas e próximas, e espere uns segundos. Elas pareiam sozinhas.

**Se a esquerda não funcionar:** a doc do ZMK sugere reiniciar central e periférico **mais ou menos ao mesmo tempo**. Aperte reset uma vez em cada metade, quase juntas.

---

## 6. Parear com o Mac

1. **Ajustes do Sistema → Bluetooth.**
2. Deve aparecer um dispositivo chamado **"roBa"**. Clique em **Conectar**.
3. Teste digitando.

**Atenção ao cabo:** pela doc do ZMK, quando USB e Bluetooth estão conectados ao mesmo tempo, a saída vai por padrão para o **USB**. Se a metade direita estiver no cabo ligado ao Mac, o teclado vai funcionar pelo cabo, não pelo Bluetooth. Isso é útil como plano B, mas lembre na hora de testar.

---

## 7. Checklist de teste

- [ ] Todas as letras das duas metades
- [ ] P2 (Numbers), P5 (Symbols), P2+P5 (System)
- [ ] Trackball move o cursor; P2+V clica
- [ ] **O teste que importa:** usar normalmente por alguns dias. Tecla que não sai, tecla repetindo sozinha ou layer presa não devem mais aparecer
- [ ] Segurar e soltar P2 e P5 muitas vezes seguidas. As layers não devem ficar presas
- [ ] Encoder (se tiver) gira no sentido esperado

---

## 8. ZMK Studio

Agora o Studio conversa com a **metade direita**:

1. Cabo USB na **metade direita**.
2. Abra [zmk.studio](https://zmk.studio/) no Chrome → **USB**.

O `roBa_R.conf` dessa branch já tem `CONFIG_ZMK_STUDIO=y` e `CONFIG_ZMK_STUDIO_LOCKING=n`, então não precisa do gesto de destravar.

---

## 9. O que muda no keymap

O `roBa.keymap` da `bt-zmk-0.3` é um snapshot de junho, mais antigo que o `roBakesb.keymap` que você usa hoje.

**Layers (11):** Mac, Win, Gaming, Number, Symbol, Super (= System), Mouse, Tiling, TWin, Scroll, Snipe. **Não existem** Game Num, Game Fn, Fast Scroll e H-Scroll.

**Continua igual:**

- Numbers + Symbols juntos → Super/System (conditional layer)
- Homerow mods, Tile Mac em A e `;`
- Combo `,` `.` `/` + tecla abaixo do Z → alterna a layer Mouse

**Diferente:**

| Posição | Hoje (`roBakesb`) | `bt-zmk-0.3` |
|---|---|---|
| Abaixo do Z | `flush_mods` | `&sys_reset` (removido no passo 3.2) |
| Abaixo do C | Ctrl+↑ | Cmd+Ctrl+Espaço (emojis) |
| P1 | `&mo MOUSE` | `MO_TOG(MOUSE)` (corrigido no passo 3.2) |
| Tecla abaixo da `/` | Ctrl+CapsLock | `MO_TOG(MOUSE)`: tocar rápido **trava** a Mouse |
| Cluster Lariska | 5 botões de mouse | Não existe |

**Bluetooth agora importa.** Na layer Super/System:

| Tecla | Ação |
|---|---|
| Z / X / C | Perfil Bluetooth 0 / 1 / 2 (e volta para Mac) |
| V / B | Perfil 3 / 4 (e troca para a layer **Win**) |
| M | Volta para a layer Mac |
| `.` | **`BT_CLR`**: apaga o pareamento do perfil ativo |

---

## 10. Voltar para o setup antigo (ESB + dongle)

### Caminho 1: a partir dos backups CURRENT.UF2

1. Settings reset nas duas metades (igual às 5.1 e 5.2). Isso apaga os pareamentos Bluetooth.
2. Duplo reset na esquerda → `backup-esquerda-CURRENT.UF2`.
3. Duplo reset na direita → `backup-direita-CURRENT.UF2`.
4. Reconecte o dongle no Mac.
5. No Mac, esqueça o dispositivo "roBa" em Ajustes → Bluetooth.

### Caminho 2: recompilar

1. No GitHub, em **Actions → Build → Run workflow**, escolha a **tag `backup-esb-v0.6.1`**. Se o seletor não mostrar tags, crie uma branch a partir dela:

   ```bash
   git checkout -b volta-esb backup-esb-v0.6.1
   git push -u origin volta-esb
   ```

2. Baixe o zip `firmware` e use `roBakesb_left.uf2` e `roBakesb_right.uf2`.
3. Grave como no caminho 1: settings reset, depois esquerda e direita, depois o dongle no Mac.

⚠️ Esse commit usa o workflow da `main`, que compila as ~24 combinações ESB. Se alguma falhar e o zip `firmware` não aparecer, os arquivos individuais ficam disponíveis como `artifact-roBakesb_left` e `artifact-roBakesb_right`.

---

## 11. Problemas comuns

| Sintoma | O que fazer |
|---|---|
| O drive USB não aparece | Cabo só de carga; troque por um de dados. Ou o duplo clique foi lento demais: tem que ser dentro de ~0,5 s |
| A esquerda não responde | Reinicie as duas metades quase juntas. Se não resolver, refaça a seção 5 inteira |
| O Mac não acha "roBa" | Refaça o settings reset. Depois do reset, esqueça o teclado no Mac antes de parear de novo (doc do ZMK) |
| Conectou mas parou de funcionar | Doc do ZMK para macOS: esquecer no Bluetooth do Mac → `BT_CLR` (System + `.`) no perfil ativo → parear de novo |
| A esquerda dura muito mais que a direita | Esperado: o central gasta mais |
| Teclado "morreu" do nada | Se não aplicou o passo 3.2, pode ter apertado a tecla abaixo do Z (`sys_reset`) |

---

## Fontes

- ZMK, split keyboards (papéis, bateria, latência): <https://zmk.dev/docs/features/split-keyboards>
- ZMK, settings reset e macOS: <https://zmk.dev/docs/troubleshooting/connection-issues>
- ZMK, gravação UF2: <https://zmk.dev/docs/user-setup>
- ZMK, prioridade USB vs BLE: <https://zmk.dev/docs/keymaps/behaviors/outputs>
- Workflow oficial ZMK v0.3.0 (nomes dos artifacts, zip `firmware`): `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3.0`
- Bootloader Adafruit nRF52 (duplo reset, `CURRENT.UF2`): <https://github.com/adafruit/Adafruit_nRF52_Bootloader>
- Branch `bt-zmk-0.3`: <https://github.com/charlesmst/zmk-config-generic/tree/bt-zmk-0.3>
