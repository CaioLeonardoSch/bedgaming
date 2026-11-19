# Roadmap — BedGaming

Cada fase tem um critério de conclusão. Não avançar sem cumpri-lo.

## Fase 0 — Linha de base com o Moonlight oficial (sem código)

- Sunshine instalado e configurado no PC.
- Moonlight Tizen oficial instalado na TV (Apps2Samsung ou Tizen Studio).
- Confirmar a versão do Tizen (Moonlight → Settings → About → System information).
- Matriz de testes: HEVC e AV1 × 40/60/80 Mbps × Game Mode on/off × SDR/HDR, em 4K60.
- Controle Xbox pareado na TV.

**Concluída quando:** existe uma tabela com FPS, quadros perdidos e latências (host/rede) para cada
combinação, e sabemos se 4K60 é estável no Wi-Fi atual.

## Fase 1 — Ambiente de desenvolvimento

Passo a passo em [FASE_0_E_1.md](FASE_0_E_1.md).

**Concluída quando:** um app "hello world" e o BedGaming compilado a partir deste repositório são
instalados, executados, depurados no Web Inspector e permanecem instalados após reiniciar a TV.

## Fase 2 — Prova de conceito de mídia

- Instrumentar o caminho envio → decodificação com timestamps.
- Opcional: harness mínimo que injeta um fluxo HEVC gravado (Annex B, sem B-frames) no WASM Player,
  para medir o decodificador isolado.
- Validar 4K60, Main10/HDR, `kUltraLow` × `kLow`, sessões de 1 hora.

**Concluída quando:** sabemos o limite real do decodificador e a latência de decodificação.

## Fase 3 — Integração com streaming

- Rebranding: ID de pacote/app próprio (`BedGaming`) para conviver com o Moonlight oficial.
- Revisar pareamento, descoberta e autenticação (reaproveitados).

## Fase 4 — Controles

- Validar mapeamento Xbox, vibração, conexão/desconexão durante a sessão.

## Fase 5 — Otimização

- Ajustes de roteador (canal, largura, WMM), bitrate, codec, frame pacing.
- Métricas de jitter e perda de quadros; diagnóstico de Wi-Fi.

## Fase 6 — Polimento

- Perfis prontos ("Quarto 4K60 HEVC"), reconexão automática, Wake-on-LAN, UI simplificada,
  tela de diagnóstico.
