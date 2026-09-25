# Changelog — roBa (branch `bt-zmk-0.3`)

Mudanças da minha configuração, da mais recente para a mais antiga. A `main` segue
[charlesmst/zmk-config-generic](https://github.com/charlesmst/zmk-config-generic); esta branch é a versão
Bluetooth, com a metade direita como central e sem dongle.

Toda mudança aqui exige **regravar as duas metades**, salvo onde indicado.

---

## 2026-09-25 — Slave latency 0 no link com o host (lag do trackball)

O trackball estava arrastado por Bluetooth. Ligando a metade direita no cabo USB o lag **some por completo** —
e nesse caminho o cursor não passa por nenhum salto BLE (`sensor → central → USB → Mac`). Isso põe o problema
no link entre a central e o Mac, não no sensor nem no SPI.

O ZMK 0.3 usa, por padrão (`app/Kconfig`):

```
BT_PERIPHERAL_PREF_MIN_INT   6     # 7,5 ms
BT_PERIPHERAL_PREF_MAX_INT  12     # 15 ms
BT_PERIPHERAL_PREF_LATENCY  30     # <-- pode pular 30 eventos seguidos
BT_PERIPHERAL_PREF_TIMEOUT 400
```

O intervalo já é bom. A *slave latency* 30 é que permite à metade direita ignorar até 30 eventos de conexão
seguidos — a 7,5–15 ms, de 225 a 450 ms de silêncio autorizado. Faz sentido para um teclado, que trabalha em
rajadas; para um dispositivo de apontamento, é suspeito. Daí `CONFIG_BT_PERIPHERAL_PREF_LATENCY=0` no
`roBa_R.conf`.

**Custo:** bateria. Com latency 0 o rádio acorda em todo evento de conexão.

**Hipótese, não fato documentado.** Em teoria um periférico com dado pendente transmite no evento seguinte
independente da latency. Não encontrei confirmação disso em doc oficial — o que sustenta a mudança é o teste
do USB, que é forte.

**Por que não a correção do upstream.** A `main` resolve isso com
`CONFIG_BT_CTLR_CONN_INTERVAL_LOW_LATENCY=y` (comentado lá como *"Issue #3381: sub-7.5ms low-latency BLE split,
central side"*), mas esse símbolo **não existe** no `sdk-zephyr v3.5.99-ncs1`, que é o que este build usa —
conferido no repositório da Nordic. Ele só chega junto com a migração de base.

## 2026-09-25 — Setas em T invertido e Cmd+espaço removido

**Layer Number, mão direita.** As setas estavam em fila (`← ↓ ↑ →` em H J K L, estilo vim) e nunca entraram
na memória muscular. Passaram para T invertido, que é o arranjo que todo mundo já conhece:

```
        ANTES                              DEPOIS
 H    J    K    L    ;             H      J     K    L      ;
 ←    ↓    ↑    →   BSPC          Home  PgDn    ↑   PgUp   BSPC

 N    M    ,    .    /             N      M     ,    .      /
Home PgDn PgUp End  DEL           End     ←     ↓    →     DEL
```

O `↑` ficou exatamente acima do `↓`. Home / End / PgUp / PgDn assumiram as posições liberadas na linha de cima.

**Custo:** três das quatro setas desceram da home row, então navegar muito exige mover a mão. Em troca, acerta-se
de primeira. E as teclas de navegação, menos usadas, passaram a ocupar a linha mais confortável — o inverso do
ideal, mas consequência inevitável de descer as setas.

**Layer Mac.** A tecla à direita do `B` era `Cmd+espaço` (Spotlight) e virou `&none`. O atalho continua
disponível segurando `F` (é `LGUI` no hold) + o espaço do polegar.

## 2026-09-25 — Deep sleep de 5 para 30 minutos

`CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` `300000` → `1800000` nos dois `.conf`. Com o wake finalmente funcionando
(ver abaixo), os 5 minutos deixaram de ser proteção e viraram só incômodo — o teclado dormia no intervalo de
um café. 30 min é o mesmo valor que o shield `roBakesb` usa lá no upstream.

## 2026-09-24 — Correção do deep sleep: QSPI desabilitado

**O sintoma.** Passados os 5 minutos de inatividade, o teclado morria: nenhuma tecla de nenhuma das metades o
trazia de volta, o Mac mostrava o teclado pareado mas não conectado, e a única saída era cortar a energia na
chave física (ou plugar o USB). Bateria em 47%, então não era brownout.

**A causa.** O overlay de board do próprio ZMK (`app/boards/seeeduino_xiao_ble.overlay`) habilita `&qspi` com a
flash externa `p25q16h` em todo build. Nada na configuração usa esse chip — a placa define
`chosen { zephyr,flash = &flash0; }` e as settings vivem na flash interna via NVS — mas ele é registrado como
device de PM.

Ao entrar em deep sleep, `app/src/activity.c` chama `zmk_pm_suspend_devices()` e **aborta se qualquer device
falhar o suspend**:

```c
set_state(ZMK_ACTIVITY_SLEEP);            // BLE ja derrubado
if (zmk_pm_suspend_devices() < 0) {
    zmk_pm_resume_devices();
    return;                                // nunca chega no sys_poweroff()
}
sys_poweroff();
```

O `qspi_nor_pm_action()` do Zephyr retorna `-EBUSY` em três situações e, com `CONFIG_PM_DEVICE_RUNTIME`
desligado, ainda chama `qspi_device_init()` no caminho de SUSPEND e propaga o erro dele. Resultado: o teclado
anunciava o sleep, derrubava o Bluetooth e **nunca desligava** — ficava preso, tentando de novo a cada segundo.

**A correção.** `&qspi { status = "disabled"; }` no `roBa.dtsi`, que as duas metades incluem. Confirmado na
prática: depois de 7 minutos parado, uma tecla da direita traz o teclado de volta.

**Alternativa não usada:** `CONFIG_PM_DEVICE_RUNTIME=y` resolveria o mesmo ponto (e faria o
`zmk_pm_suspend_devices()` pular tanto o QSPI quanto o kscan), mas mexe no PM de todos os devices. Fica como
plano B se algo relacionado reaparecer.

## 2026-09-23 — Documentação e explorador do keymap

`docs/` com o explorador interativo (GitHub Pages), o guia de migração e o histórico do firmware ESB.
Tudo vive só nesta branch, então não conflita com o upstream.

## 2026-09-23 — Limpeza do keymap: 12 → 7 layers

Removidas as layers `Win`, `Gaming`, `Gaming Num`, `Tiling` e `TWin`, que não são usadas neste setup.
Layers finais: `Mac 0`, `Number 1`, `Symbol 2`, `Super 3`, `Mouse 4`, `Scroll 5`, `Snipe 6`.

Junto:

- Removidos os gatilhos acidentais da layer Mouse: o `MO_TOG` abaixo do `/` (alcançável de qualquer layer por
  transparência) e o combo `,` + `.` + `/` + tecla abaixo do `Z`. A Mouse passou a ter uma entrada só: segurar P1.
- `roBa.dtsi`: os *input processors* do trackball apontavam para as layers 10 e 11. Com a renumeração,
  `scroll { layers = <5>; }` e `snipe { layers = <6>; }`. **Se renumerar as layers de novo, esse arquivo tem que
  acompanhar** — senão scroll e snipe param de funcionar sem nenhum erro de compilação.
- Layer Super: perfis Bluetooth reduzidos de 5 para 3 (`Z`, `X`, `C`), `BT_CLR` no `.`, e a tecla `TO Mac`
  removida por ser redundante.

## 2026-09-22 — Migração do dongle ESB para Bluetooth

Metade direita passou a ser a central BLE; o dongle saiu de cena. O `build.yaml` compila
`roBa_R` (central, com o snippet `studio-rpc-usb-uart` para o ZMK Studio), `roBa_L_enc_inv` (periférica) e
`settings_reset`.

Motivo: com ESB, o transmissor desiste depois de N retransmissões e o pacote se perde. O BLE retransmite até
receber ACK. As falhas intermitentes de tecla acabaram.

---

## Notas que valem para qualquer mudança futura

- **Grave sempre as duas metades** quando mexer em `roBa.dtsi` ou nos `.conf` — o `.dtsi` é compartilhado.
- **Não grave o `settings_reset.uf2`** por engano: ele apaga os pareamentos Bluetooth.
- **Guarde o `firmware.zip`** de cada versão que funcionar. Os artifacts do GitHub Actions expiram (90 dias por
  padrão), e é o caminho de volta mais rápido quando algo dá errado.
- **Com USB plugado o teclado não entra em deep sleep** (`!is_usb_power_present()` em `activity.c`) — então não
  dá para investigar problemas de sleep com log por USB.
