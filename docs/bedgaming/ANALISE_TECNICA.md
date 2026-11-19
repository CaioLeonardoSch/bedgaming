# Análise técnica — BedGaming

Data da análise: novembro de 2026. Legenda: **[F]** fato confirmado em fonte; **[H]** hipótese a validar
na TV.

## 1. Plataforma

- **[F]** S90D é modelo 2024, lançado com **Tizen 8.0**. Nas tabelas de mídia da Samsung pertence ao
  grupo **24TV_PREMIUM4**.
- **[H]** A Samsung oferece upgrades de sistema para modelos recentes; a TV pode receber (ou já ter
  recebido) Tizen 9.0. Ver risco R1.
- **[F]** Apps de terceiros em TVs Samsung são **Tizen Web Apps** (`.wgt`: HTML/JS, com módulos
  WebAssembly opcionais). **[H]** Não há caminho público para apps nativos C/C++ em TVs de consumo.
- **[F]** JavaScript padrão não abre sockets UDP/TCP crus. O protocolo do Moonlight usa UDP (vídeo,
  áudio, controle via ENet). Portanto é obrigatório **WASM + Tizen Sockets Extension** (API da Samsung,
  disponível apenas em *worker threads*; `poll()` recomendado no lugar de `select()`; sem `fcntl()`,
  usar `SOCK_NONBLOCK`).

## 2. Mídia

### Por que não AVPlay / MSE

- AVPlay é orientado a URL (arquivo, HLS, DASH) com buffering de VOD. Não aceita quadros de um
  protocolo próprio com latência baixa.
- MSE do navegador também faz buffering e não resolve a falta de UDP.

### Tizen WASM Player (`samsung::wasm::ElementaryMediaStreamSource`)

- **[F]** O app entrega *elementary packets* (NAL units já separadas) e a TV decodifica por hardware,
  exibindo num `HTMLMediaElement`.
- **[F]** Modos de latência: `kNormal`, `kLow`, `kUltraLow`. `kUltraLow` desativa realces de imagem e
  ativa Game Mode. Disponibilidade por aparelho via `EmssVersionInfo::has_ultra_low_latency`.
- **[F]** Modos de renderização: `kMediaElement` (recomendado pela Samsung para game streaming, menor
  latência) e `kVideoTexture` (WebGL).
- **[F]** Restrições nos modos de baixa latência: **sem B-frames**, sem DRM, sem seek, duração
  infinita, relógio controlado pelo app.
- **[F]** O Moonlight Tizen usa `kUltraLow` quando "Game Mode" está ligado, senão `kLow`, sempre com
  `kMediaElement` (`wasm/main.cpp`).

### Codecs — grupo 24TV_PREMIUM4 (tabela oficial 2024)

| Codec | Limite oficial | Observação |
|---|---|---|
| HEVC Main | 4096×2160 @ 60 fps, 80 Mbps; nível UHD 5.2 | Codec principal recomendado |
| AV1 | 3840×2160 @ 120 fps, 40 Mbps | RTX 5070 codifica AV1; comparar com HEVC |
| H.264 | 4K60 na tabela | O Moonlight Tizen declara nível 4.2 (`avc1.64002A`): na prática, até 1080p |
| HEVC Main10 | não aparece na linha do PREMIUM4 | **[H]** provavelmente funciona (TV HDR10+); testar |

- **[H]** As tabelas descrevem reprodução de arquivos. Os limites reais em `kUltraLow` só se confirmam
  em teste (Fase 2).
- O Moonlight Tizen declara HEVC como `hev1.1.6.L153.B0` (nível 5.1) e AV1 como `av01.0.13M.08`.

### Frequência

- **[F]** Documentado pelo Moonlight Tizen: apps Tizen tendem a ficar limitados a ~60 FPS de
  renderização; 120 FPS é experimental; a opção de 144 FPS foi removida (falhava ao iniciar).
- **[F]** Há relatos de engasgos porque a TV não inicializa a taxa máxima de atualização. Contorno
  relatado: ligar um dispositivo HDMI antes de abrir o app.
- Decisão: **meta 4K/60**; 120 FPS fica para investigação futura.

### Áudio

- **[F]** Estéreo PCM é o mais confiável. 5.1/7.1 em PCM frequentemente falha em apps Tizen.
- **[F]** Opus é decodificado em software (libopus no WASM) e entregue como PCM.

## 3. Controles

- **[F]** API Gamepad padrão W3C (`navigator.getGamepads`, `gamepadconnected`), vibração via
  `vibrationActuator.playEffect("dual-rumble")`, até 4 controles.
- **[F]** O pareamento Bluetooth é feito **no menu da TV** (Configurações → Conexão → Gerenciador de
  dispositivos externos → Gerenciador de dispositivos de entrada). O app apenas lê o gamepad.
- **[F]** Controles Xbox One/Series são os mais recomendados pela comunidade.

## 4. Rede

- **[F]** A porta Ethernet de muitas TVs Tizen é de 100 Mbps; **Wi-Fi 5 GHz costuma ser melhor** que o
  cabo para streaming em 4K.
- **[H]** A 5–10 m com uma parede, um Wi-Fi 5/6 bem configurado sustenta HEVC 4K60 a 40–80 Mbps. O
  problema real é **jitter**, não banda.
- Roteador TP-Link XX530v (**[H]** Wi-Fi 6 dual-band). Ajustes previstos: SSID 5 GHz dedicado, canal
  fixo, largura 80 MHz, WMM ligado, TV esquecida na rede 2.4 GHz.

## 5. Ecossistema Sunshine/Moonlight — reaproveitamento

| Componente | Origem | Licença | Decisão |
|---|---|---|---|
| Protocolo (RTSP, ENet criptografado, RTP + FEC, Opus, input) | `moonlight-common-c` | GPL-3.0 | Reaproveitar |
| Pareamento/HTTPS com o host | `libgamestream` | GPL-3.0 | Reaproveitar |
| Mídia, sockets, áudio (WASM) | `wasm/*.cpp` | GPL-3.0 | Reaproveitar |
| UI de TV, gamepad, configurações | `wasm/platform/*.js` | GPL-3.0 | Reaproveitar e evoluir |
| Build reproduzível | `Dockerfile` (Tizen CLI 6.1 + Emscripten Samsung 1.39.4.7 fastcomp) | — | Reaproveitar |

- Sunshine (LizardByte) ativo em 2026, versões numeradas por data; houve atualização crítica de
  segurança em maio de 2026. RTX 5070 com NVENC atende HEVC e AV1.
- GPL-3.0: uso pessoal sem restrição. Se o app for distribuído, o código-fonte deve acompanhar.

## 6. Arquitetura

```
PC (RTX 5070, Ethernet)                     Samsung S90D (Wi-Fi 5 GHz)
┌─────────────────────┐                    ┌───────────────────────────────────────┐
│ Jogo                │                    │ BedGaming (.wgt)                       │
│ Sunshine            │  HTTPS 47984/47989 │  UI JS (TV + gamepad W3C)              │
│  captura → NVENC    │◄──── pareamento ──►│  WASM (worker):                        │
│  HEVC/AV1, sem      │  RTSP / ENet / UDP │   moonlight-common-c + Tizen Sockets   │
│  B-frames           │═══ vídeo/áudio ═══►│   → ElementaryMediaStreamSource        │
│                     │◄═══ input ════════ │     kUltraLow/kLow + kMediaElement     │
└─────────────────────┘                    │   → decodificador HW → painel          │
                                           └───────────────────────────────────────┘
```

Orçamento de latência estimado **[H]**: 30–60 ms ponta a ponta (captura+encode 3–6 ms, Wi-Fi 2–10 ms,
decode 10–25 ms, exibição em Game Mode). O app já exibe latência de host e de rede nas estatísticas.

## 7. Fluxo de desenvolvimento

- **[F]** Ferramentas: Tizen Studio 6.1 + *TV Extensions* + *Samsung Certificate Extension*
  **≥ 2.0.73** (versões anteriores não criam certificados após setembro de 2025).
- **[F]** Certificado Samsung: perfil = certificado de autor + distribuidor; exige conta Samsung e o
  DUID da TV. Atualizações do app devem ser assinadas pelo **mesmo** certificado de autor.
- **[F]** Developer Mode: Apps → digitar `12345` → ligar → IP do PC → reiniciar a TV. Só o IP
  informado consegue conectar via `sdb`.
- **[F]** Instalação via USB não é mais suportada pela Samsung.
- **[F]** Depuração: Web Inspector (DevTools remoto) via Tizen Studio "Debug As" ou
  `sdb shell 0 debug <appId>`.
- **[H]** A documentação oficial não informa validade do certificado nem se o Developer Mode sobrevive
  a atualizações de firmware. Pela experiência da comunidade, o app permanece após reinicializações
  normais.
- **[H]** O `Dockerfile` assina com um certificado Tizen genérico. Em Tizen 7+ pode ser necessário
  reassinar com o perfil Samsung antes de instalar (validar na Fase 1).

## 8. Riscos

| # | Risco | Mitigação |
|---|---|---|
| R1 | Upgrade para Tizen 9 quebra Game Mode (tela preta, congelamento, corrupção > 1080p) | Variante ForceGM; considerar congelar atualizações automáticas depois que tudo funcionar |
| R2 | Firmware remove o app ou invalida o certificado | Backup do `.p12` e senha; build reproduzível |
| R3 | Toolchain congelada (Emscripten 1.39.4.7 fastcomp, 2021) | Isolar no Docker; não atualizar sem motivo |
| R4 | Upstream encerra desenvolvimento ativo no fim de 2026 | Manter o fork e acompanhar `upstream` |
| R5 | Limites reais do decodificador em `kUltraLow` 4K | Medir na Fase 2 |
| R6 | Jitter do Wi-Fi | Ajustes do roteador; medir na Fase 5 |
| R7 | Certificado/Developer Mode (validade, DUID) | Documentar renovação na Fase 1 |

## Fontes

- Moonlight Tizen: repositório e wiki — https://github.com/brightcraft/moonlight-tizen
- Especificações de vídeo 2024 — https://developer.samsung.com/smarttv/develop/specifications/media-specifications/2024-tv-video-specifications.html
- Grupos de modelos — https://developer.samsung.com/smarttv/develop/specifications/tv-model-groups.html
- Tizen WASM Player — https://developer.samsung.com/smarttv/develop/extension-libraries/webassembly/tizen-wasm-player/overview.html
- Game Streaming on Tizen TV with Wasm — https://developer.samsung.com/smarttv/develop/extension-libraries/webassembly/game-streaming-on-tizen-tv-with-wasm.html
- Tizen Sockets Extension — https://developer.samsung.com/smarttv/develop/extension-libraries/webassembly/api-reference/tizen-sockets-extension.html
- Certificados — https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/creating-certificates.html
- Conexão com a TV — https://developer.samsung.com/smarttv/develop/getting-started/using-sdk/tv-device.html
- Instalação do SDK — https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/installing-tv-sdk.html
- Gamepad — https://developer.samsung.com/smarttv/develop/guides/user-interaction/gamepad.html
- Sunshine — https://github.com/LizardByte/Sunshine
