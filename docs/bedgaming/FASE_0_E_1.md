# Fase 0 e Fase 1 — Passo a passo (Windows 11)

Substitua `IP_DA_TV`, `IP_DO_PC` e `ID_DO_DISPOSITIVO` pelos valores reais. Não commite esses valores.

## Preparação da rede (faça primeiro)

1. Descubra o IP do PC: no PowerShell, `ipconfig` → adaptador Ethernet → "Endereço IPv4".
2. No roteador (TP-Link XX530v), crie uma **reserva de DHCP** para o PC e para a TV. O Developer Mode
   só aceita conexões do IP informado na TV; se o IP do PC mudar, a conexão para de funcionar.
3. Na TV, conecte apenas à rede **5 GHz** e esqueça a rede 2.4 GHz.
4. Descubra o IP da TV: Configurações → Geral → Rede → Status da rede → Configurações de IP.

## Fase 0 — Linha de base

### 0.1 Sunshine no PC

1. Baixe o instalador oficial em https://github.com/LizardByte/Sunshine/releases (versão estável mais
   recente, não *pre-release*).
2. Instale, abra a interface web (`https://localhost:47990`) e crie o usuário administrador.
3. Libere o Sunshine no Firewall do Windows para **redes privadas** (o instalador costuma fazer isso).
4. Em Configuration → Video, confirme o encoder **NVENC**.

### 0.2 Developer Mode na TV

1. Abra **Apps**. Selecione **Configurações do app** (App Settings) e digite `1 2 3 4 5` no controle
   remoto (use o botão `123` para o teclado numérico na tela, se necessário).
2. Ligue **Developer mode** e informe `IP_DO_PC` em "Host PC IP". Confirme.
3. Reinicie a TV **segurando o botão de energia por 5 segundos** até aparecer a tela de boot.
4. Ao abrir **Apps**, deve aparecer a marca **DEVELOP MODE** no topo.

### 0.3 Instalar o Moonlight oficial

Opção mais simples: [Apps2Samsung](https://github.com/Apps2Samsung/Apps2Samsung), seguindo o
[guia do Moonlight Tizen](https://github.com/brightcraft/moonlight-tizen/wiki/Installation-Guide#installation-using-apps2samsung).
Em Tizen 7+ ele pedirá login na conta Samsung para assinar o pacote. Use a variante padrão
(`Moonlight-Tizen`); a ForceGM é só para Tizen 9+.

### 0.4 Primeiros testes

1. Abra o Moonlight, adicione o PC, digite o PIN no Sunshine.
2. Anote a versão do Tizen em Settings → About → System information.
3. Pareie o controle Xbox na TV: Configurações → Conexão → Gerenciador de dispositivos externos →
   Gerenciador de dispositivos de entrada → Lista de dispositivos Bluetooth.
4. Na TV, use o modo de imagem com Game Mode e desligue processamentos (Picture Clarity, etc.).
5. Teste cada combinação em 4K60 com estatísticas ligadas, por ~5 minutos:

| Codec | Bitrate | Game Mode | HDR | FPS médio | Quadros perdidos | Latência host | Latência rede | Observações |
|---|---|---|---|---|---|---|---|---|
| HEVC | 40 | on | off | | | | | |
| HEVC | 60 | on | off | | | | | |
| HEVC | 80 | on | off | | | | | |
| AV1 | 40 | on | off | | | | | |
| HEVC | 60 | off | off | | | | | |
| HEVC | 60 | on | on | | | | | |

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
