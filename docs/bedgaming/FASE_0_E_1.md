# Fase 0 e Fase 1 — Passo a passo (Windows 11)

Substitua `IP_DA_TV`, `IP_DO_PC` e `ID_DO_DISPOSITIVO` pelos valores reais. Não commite esses valores.

## Preparação da rede (faça primeiro)

1. Descubra o IP do PC: no PowerShell, `ipconfig` → adaptador **Ethernet** → "Endereço IPv4".
   Ignore o adaptador `vEthernet (Default Switch)` (rede virtual do Hyper-V/WSL, faixa `172.x`).
2. Confira o **perfil de rede** da Ethernet: Configurações → Rede e Internet → Ethernet → Perfil de
   rede → **Privada**. Em "Pública" o Windows bloqueia mais e a TV pode não achar o PC.
3. No roteador (TP-Link Aginet XX530v, painel em `http://tplinkmodem.net`), crie uma **reserva de
   DHCP** para o PC e para a TV. O Developer Mode só aceita conexões do IP informado na TV; se o IP
   do PC mudar, a conexão para de funcionar.
   - O painel **não** usa a senha da etiqueta (essa é a do Wi-Fi de fábrica). A senha de
     administrador é criada no primeiro acesso; se foi o técnico da operadora quem configurou,
     peça a ele/ao suporte. Após várias senhas erradas o painel bloqueia por 600 s.
4. Na TV, conecte apenas à rede **5 GHz** e esqueça a rede 2.4 GHz. Confira em Configurações →
   Geral → Rede → Status da rede. Na 2.4 GHz o controle Bluetooth fica com atraso grande (ver registro
   abaixo).
5. Descubra o IP da TV: Configurações → Geral → Rede → Status da rede → Configurações de IP.

## Fase 0 — Linha de base

### 0.1 Sunshine no PC

1. Instale a versão estável: `winget install --id LizardByte.Sunshine -e` (ou o `.msi` de
   https://github.com/LizardByte/Sunshine/releases, não *pre-release*). Aceite o UAC.
2. O instalador cria o serviço `SunshineService` (início automático) e regras de firewall de
   entrada TCP/UDP para `sunshine.exe` em todos os perfis.
3. Abra `https://localhost:47990` (aceite o aviso de certificado próprio) e crie o usuário
   administrador. Guarde a senha num cofre de senhas.
4. NVENC é escolhido automaticamente: o log (`C:\Program Files\Sunshine\config\sunshine.log`) deve
   listar `h264_nvenc`, `hevc_nvenc` e `av1_nvenc`. O erro `gpu doesn't support YUV444 encode`
   durante o teste de encoders é esperado e pode ser ignorado.
   - A lista "NVIDIA NVENC / Intel QuickSync / AMD AMF / Software" no menu da interface são só
     páginas de configuração; os textos em cinza nos campos são exemplos, não valores.

### 0.1b Drivers auxiliares no PC

**Controle virtual (obrigatório para jogar com o controle):**

- `winget install --id ViGEm.ViGEmBus -e` (ViGEmBus 1.22.0, gratuito; projeto arquivado, mas
  suportado pelo Sunshine; emula Xbox 360/DualShock 4).
- A alternativa "Virtual HID Broker" exige licença paga.
- No Sunshine: Configuration → **Input** → Gamepad driver → **ViGEmBus** (`gamepad_driver = vigembus`).
- Durante o streaming, o Windows mostra "Controlador XBOX 360 para Windows".

**Tela virtual 4K (necessária se o monitor do PC não for 3840×2160 16:9):**

- `winget install --id VirtualDrivers.Virtual-Display-Driver -e`. Isso instala só o app
  **VDD Control**; abra-o e clique em **Install Driver** (UAC). A configuração padrão
  (`C:\VirtualDisplayDriver\vdd_settings.xml`) já inclui 3840×2160 com 60 Hz.
- No log do Sunshine, após reiniciá-lo (Troubleshooting → Restart), procure o bloco com
  `"friendly_name": "VDD by MTT"` e copie o `device_id`.
- Sunshine → Configuration → **Audio/Video**:
  - **Display Id** (`output_name`): campo de texto livre; cole o `device_id` da tela virtual.
  - **Advanced display device options → Device configuration**: "Deactivate other displays and
    activate only the specified display" (`ensure_only_display`).
  - Resolução e refresh: usar os valores do cliente (padrão `auto`).
  - **Save** e **Apply**.
- Fora do streaming a tela virtual pode ficar desativada (Win+P → "Somente tela do PC"); o Sunshine
  a reativa no início da sessão e restaura o monitor físico no fim.

### 0.2 Developer Mode na TV

1. Abra a tela de **Apps** e entre em **Configurações do app** (ícone de engrenagem/"Configurações",
   no fim da aba Aplicativos). Digitar o código na vitrine "Aplicativos" da tela inicial **não**
   funciona.
2. Digite `1 2 3 4 5`. O controle SolarCell não tem números: use o botão `123` para abrir o
   teclado numérico na tela, ou o controle virtual do app SmartThings.
3. Ligue **Developer mode** e informe `IP_DO_PC` em "Host PC IP". Confirme.
4. Reinicie a TV **segurando o botão de energia por ~5 segundos** até aparecer a tela de boot.
5. Verificação no PC: `Test-NetConnection IP_DA_TV -Port 26101` deve retornar `True`.

### 0.3 Instalar o Moonlight oficial

1. `winget install --id Apps2Samsung.Apps2Samsung -e` (ou
   [Apps2Samsung](https://github.com/Apps2Samsung/Apps2Samsung)).
2. Abra o Apps2Samsung; a TV deve aparecer como "Pronto para uso".
3. Escolha **Moonlight** → variante **brightcraft · Tizen: Moonlight-Tizen.wgt**. Não use ForceGM
   (só Tizen 9+) nem as variantes de terceiros. Deixe **"Instalar com um novo ID de pacote"
   desligado** (só serve para o erro `[118, -22]`).
4. Em Tizen 7+ ele pede login na conta Samsung para gerar o certificado e assinar o pacote.
5. Se aparecer o aviso **"Redes diferentes"** citando um IP `172.x` como "this device", clique em
   **Continuar**: o app leu o adaptador virtual do Hyper-V. Confirme com
   `Find-NetRoute -RemoteIPAddress IP_DA_TV` que a rota sai pela Ethernet com `IP_DO_PC`.

### 0.4 Primeiros testes

1. Abra o Moonlight, adicione o PC e digite o PIN no Sunshine (página **PIN**).
2. Anote a versão do Tizen em ⚙ → About → System information.
3. Pareie o controle Xbox na TV: Configurações → Conexão → Gerenciador de dispositivos externos →
   Gerenciador de dispositivos de entrada → Bluetooth. No controle: ligue no botão Xbox e segure o
   botão de pareamento (em cima, perto do USB) ~3 s até o logo piscar rápido.
4. Na TV, use o modo de imagem com Game Mode e desligue processamentos (Picture Clarity, etc.).
5. Configure o Moonlight (⚙; nomes em pt-BR com o original em inglês):

| Categoria | Opção | Valor |
|---|---|---|
| Configurações Básicas (Basic Settings) | Resolução de vídeo (Video resolution) | 3840 x 2160 (4K) |
| | Taxa de quadros de vídeo (Video frame rate) | 60 FPS |
| | Taxa de bits de vídeo (Video bitrate) | conforme a matriz (controle deslizante, ◀ ▶) |
| Configurações do Host (Host Settings) | **Otimizar configurações do jogo** (Optimize game settings) | **ligado**: sem isso o Sunshine não muda a resolução da tela virtual |
| Configurações de Vídeo (Video Settings) | Codec de vídeo (Video codec) | HEVC ou AV1 |
| | **Modo de jogo** (Game mode) | **ligado**: vem desligado por padrão |
| | Video HDR | conforme a matriz |
| Configurações Avançadas (Advanced Settings) | Estatísticas de desempenho (Performance statistics) | ligado |
| | Otimizar predefinições de taxa de bits (Optimize bitrate presets) | desligado |

6. Atalhos durante o streaming:

| Ação | Controle Xbox | Controle remoto / teclado |
|---|---|---|
| Encerrar a sessão | segurar **View + Menu + LB + RB** | botão vermelho (no SolarCell, via `123`) / Ctrl+Alt+Shift+Q |
| Mostrar/ocultar estatísticas | **View + LB + RB + X** | botão amarelo / Ctrl+Alt+Shift+S |

   Pelo PC, a sessão também pode ser encerrada em `https://localhost:47990` → Troubleshooting →
   **Force Close**.

7. Teste cada combinação em 4K60 com estatísticas ligadas, por ~5 minutos, **com conteúdo em
   movimento** (jogo ou vídeo 4K60). Em área de trabalho parada o Sunshine só envia quadros quando a
   tela muda, então FPS ~20 e bitrate ~1 Mbps são normais e não medem nada.

| Codec | Bitrate | Game Mode | HDR | FPS médio | Quadros perdidos | Latência host | Latência rede | Observações |
|---|---|---|---|---|---|---|---|---|
| HEVC | 40 | on | off | | | | | |
| HEVC | 60 | on | off | | | | | |
| HEVC | 80 | on | off | | | | | |
| AV1 | 40 | on | off | | | | | |
| HEVC | 60 | off | off | | | | | |
| HEVC | 60 | on | on | | | | | |

### Registro da execução — 19/11/2026

Ambiente confirmado:

| Item | Resultado |
|---|---|
| Sistema da TV | **[F] Tizen 8** (Moonlight → About). ForceGM não é necessário |
| Moonlight | brightcraft v1.20.0, instalado via Apps2Samsung 3.3.1 |
| Sunshine | 2026.914.233613 (winget), NVENC H.264/HEVC/AV1 detectados |
| Drivers | ViGEmBus 1.22.0; Virtual Display Driver 25.7.23 |
| Monitor do PC | ultrawide 3440×1440 a 165 Hz (21:9), por isso a tela virtual |
| Roteador | [F] TP-Link Aginet XX530v, AX3000 Dual Band **Wi-Fi 6** XPON |
| Rede | ping PC → TV de 2–9 ms pelo Wi-Fi 5 GHz |

Primeira sessão válida (área de trabalho parada, HEVC, 3840×2160 capturado a 60 Hz pela tela
virtual; Game Mode ligado):

| Medida | Valor |
|---|---|
| Quadros perdidos pela rede | 0,00% |
| Latência média da rede | 1–4 ms (variância 1–4 ms) |
| Latência de processamento do host (mín / máx / média) | 4,1 / 4,2–7,5 / 4,1–5,1 ms |
| Tempo médio de decodificação | 1,3–1,8 ms |
| Tempo médio de renderização | 0,5–0,8 ms |
| Controle | reconhecido no PC como "Controlador XBOX 360 para Windows" |

O que foi diferente do previsto:

1. A rede Ethernet estava com perfil **Público**; foi mudada para Privada.
2. A senha de administrador do roteador não é a da etiqueta, e foi definida pelo instalador da
   operadora. **Reserva de DHCP ainda pendente**; por enquanto os IPs são os dados pelo DHCP.
3. O código `12345` só funciona dentro de "Configurações do app", não na vitrine "Aplicativos".
4. O Apps2Samsung mostrou o aviso "Redes diferentes" por causa do adaptador do Hyper-V (falso alarme).
5. O monitor ultrawide exigiu tela virtual. Sem ela a TV recebia 21:9 com faixas; com ela, mas sem
   "Otimizar configurações do jogo", recebia 800×600 (4:3) esticado (`"resolution": null` no log).
6. "Modo de jogo" e "Otimizar configurações do jogo" vêm **desligados** no Moonlight.
7. O controle remoto SolarCell não tem botões coloridos físicos e o Play/Pause não encerrou a sessão;
   use a combinação do controle Xbox.
8. **A TV estava conectada na rede 2.4 GHz**, não na 5 GHz. Jogando (Steam Big Picture) havia atraso
   grande no controle e o aviso "conexão lenta com o PC" aparecia, enquanto o teclado ligado ao PC
   respondia normalmente (ou seja, o vídeo estava rápido). [H] Causa provável: o Bluetooth do controle e
   o Wi-Fi 2.4 GHz disputando a mesma faixa na TV. **[F] Após mudar a TV para a rede 5 GHz, o atraso
   sumiu com HEVC 40 Mbps.** O IP da TV não mudou.

Pendente na Fase 0:

- [ ] Matriz de testes acima com jogo em movimento.
- [ ] Reserva de DHCP para PC e TV.
- [ ] Decidir sobre desligar a atualização automática de software da TV (risco R1).

## Fase 1 — Ambiente de desenvolvimento

### 1a. Tizen Studio e extensões

1. Baixe o Tizen Studio 6.1 em https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/installing-tv-sdk.html
   e instale (por exemplo, em `C:\tizen-studio`).
2. Abra o **Package Manager** (se não abrir, execute `C:\tizen-studio\package-manager\package-manager.exe`).
3. Na aba **Extension SDK**, instale **TV Extensions** e **Samsung Certificate Extension**
   (versão **2.0.73 ou superior**).
4. Adicione ao PATH: `C:\tizen-studio\tools` (sdb) e `C:\tizen-studio\tools\ide\bin` (tizen CLI).

### 1b. WSL2 e Docker

```powershell
wsl --install -d Ubuntu
```

Reinicie o Windows, instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/) com
backend WSL2 e confirme com `docker version`.

### 1c. Conectar à TV

```powershell
sdb connect IP_DA_TV
sdb devices
```

A última coluna de `sdb devices` é o `ID_DO_DISPOSITIVO` (algo como o código do modelo). No Tizen
Studio, a TV também deve aparecer em Tools → Device Manager.

### 1d. Certificado Samsung

1. Tizen Studio → Tools → **Certificate Manager** → `+` → **Samsung** → **TV**.
2. Nome do perfil: `BedGaming`.
3. **Create a new author certificate** → faça login com a conta Samsung → defina senha e
   **guarde o backup do `.p12` fora do repositório** (ex.: um cofre de senhas ou pasta sincronizada
   privada).
4. **Create a new distributor certificate** → privilégio **Public** → **Add individual DUIDs**: o DUID
   da TV é preenchido automaticamente se ela estiver conectada no Device Manager.
5. Finalize; o perfil `BedGaming` fica ativo.

> Se a TV for trocada ou o DUID mudar, é preciso gerar um novo distribuidor. Atualizações do app
> precisam do **mesmo** certificado de autor.

### 1e. Hello world

1. File → New → Tizen Project → **Template** → **TV-Samsung** (versão mais alta disponível) →
   **Web Application** → **Basic Project**. Nome: `HelloBed`.
2. Clique com o botão direito no projeto → **Run As → Tizen Web Application**. O app deve abrir na TV.
3. **Debug As → Tizen Web Application** deve abrir o Web Inspector no navegador.
4. Reinicie a TV (botão de energia por 5 s) e confirme que `HelloBed` continua instalado.

Pela linha de comando (alternativa):

```powershell
tizen package -t wgt -s BedGaming -- C:\caminho\HelloBed
tizen install -n HelloBed.wgt -t ID_DO_DISPOSITIVO
```

### 1f. Compilar o BedGaming a partir deste repositório

```powershell
git clone https://github.com/CaioLeonardoSch/bedgaming.git
cd bedgaming
docker build -t bedgaming .
docker create --name bg bedgaming
docker cp bg:/home/moonlight/Moonlight.wgt .
docker rm bg
```

O `.wgt` gerado no Docker é assinado com um certificado Tizen genérico. Em Tizen 7+ a TV pode
recusá-lo; nesse caso, reassine com o perfil Samsung:

```powershell
mkdir build-wgt
tar -xf Moonlight.wgt -C build-wgt
Remove-Item build-wgt\author-signature.xml, build-wgt\signature1.xml -ErrorAction SilentlyContinue
tizen package -t wgt -s BedGaming -- build-wgt
tizen install -n build-wgt\Moonlight.wgt -t ID_DO_DISPOSITIVO
```

> Enquanto o ID do pacote não for trocado (Fase 3), esta compilação **substitui** o Moonlight oficial
> instalado na Fase 0 (mesmo `MoonLightS.MoonlightWasm`). Se a assinatura for diferente, desinstale o
> oficial antes: `sdb shell 0 vd_appuninstall MoonLightS.MoonlightWasm`.

### 1g. Depuração do BedGaming

```powershell
sdb shell 0 debug MoonLightS.MoonlightWasm
```

Abra no navegador o endereço `http://IP_DA_TV:PORTA` indicado pelo comando → aba **Console**.

### Critério de conclusão da Fase 1

- [ ] `HelloBed` instalado, executado, depurado e persistente após reinicialização.
- [ ] BedGaming compilado a partir deste repositório, instalado e conectado ao Sunshine.
- [ ] Backup do certificado de autor guardado fora do repositório.
- [ ] Problemas encontrados anotados neste documento.
