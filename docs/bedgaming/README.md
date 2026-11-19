# BedGaming

Cliente pessoal de *game streaming* para a Samsung OLED S90D (Tizen), recebendo jogos de um PC com
[Sunshine](https://github.com/LizardByte/Sunshine) pela rede doméstica, com controle Bluetooth
conectado à TV.

## Origem e créditos

O BedGaming é um **fork pessoal** do [Moonlight Tizen](https://github.com/brightcraft/moonlight-tizen),
mantido por [BrightCraft](https://github.com/brightcraft) e contribuidores. O Moonlight Tizen, por sua
vez, deriva do [Moonlight Chrome](https://github.com/moonlight-stream/moonlight-chrome) (projeto
[Moonlight](https://moonlight-stream.org/)), do port WASM do
[Samsung Developers Forum](https://github.com/SamsungDForum/moonlight-chrome) e dos forks de
KyroFrCode, OneLiberty, toypoodlegaming e outros, listados no [README principal](../../README.md).

Todo o histórico de commits do projeto original foi preservado neste repositório. O remoto `upstream`
aponta para `https://github.com/brightcraft/moonlight-tizen`.

Licença: **GNU GPL v3.0**, a mesma do projeto original (ver [LICENSE](../../LICENSE)).

## Por que um fork e não um cliente do zero

A análise técnica ([ANALISE_TECNICA.md](ANALISE_TECNICA.md)) concluiu que o Moonlight Tizen já
implementa pareamento, protocolo GameStream/Sunshine, decodificação por hardware (HEVC/AV1) via
Tizen WASM Player, gamepad W3C e Game Mode. Reescrever isso levaria meses sem ganho técnico. O
trabalho próprio do BedGaming fica concentrado em medição, ajustes para o cenário "quarto, 4K60, Wi-Fi",
diagnóstico e polimento.

## Documentos

| Documento | Conteúdo |
|---|---|
| [ANALISE_TECNICA.md](ANALISE_TECNICA.md) | Plataforma, APIs, limitações, riscos, arquitetura |
| [FASE_0_E_1.md](FASE_0_E_1.md) | Passo a passo: linha de base com o Moonlight oficial e ambiente de desenvolvimento no Windows 11 |
| [ROADMAP.md](ROADMAP.md) | Fases 0 a 6 e critérios de conclusão |

## Ambiente-alvo

| Item | Valor |
|---|---|
| TV | Samsung OLED S90D 55" (modelo brasileiro 2024, grupo de mídia `24TV_PREMIUM4`) |
| Sistema da TV | Tizen 8.0 de fábrica (confirmar se já atualizou para 9.0) |
| Host | Windows 11, NVIDIA RTX 5070 12 GB, Ethernet |
| Roteador | TP-Link XX530v, rede 5 GHz dedicada para a TV |
| Meta inicial | 3840×2160, 60 FPS, HEVC (AV1 como alternativa), baixa latência |

> Dados identificadores da TV (número de série, DUID) **não** devem ser commitados neste repositório.
