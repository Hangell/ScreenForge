# 🎬 ScreenForge

<p align="center">
  <img src="./assets/screenforge-logo.png" alt="ScreenForge Logo" width="128" height="128">
  <br />
  <strong>Uma aplicação multiplataforma de gravação de tela poderosa e fácil de usar</strong>
  <br />
  Construída com Avalonia UI e FFmpeg para performance superior
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-.NET%206+-blue.svg" alt=".NET 6+" />
  <img src="https://img.shields.io/badge/UI-Avalonia-purple.svg" alt="Avalonia UI" />
  <img src="https://img.shields.io/badge/Video-FFmpeg-green.svg" alt="FFmpeg" />
  <img src="https://img.shields.io/badge/OS-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg" alt="Cross-platform" />
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT License" />
</p>

## ✨ Características

- 🖥️ **Captura de múltiplos monitores**: Grave qualquer monitor ou a área de trabalho virtual completa
- 🎯 **Preview em tempo real**: Visualização overlay da área selecionada para captura
- 📁 **Pasta personalizável**: Escolha onde salvar suas gravações
- ⚡ **Performance otimizada**: Usa codificadores de hardware quando disponíveis (h264_videotoolbox, h264_mf)
- 🌐 **Multiplataforma**: Windows, macOS e Linux
- 🎵 **Suporte a áudio**: Captura de áudio opcional (configurável)
- 💾 **Auto-save**: Salvamento automático com timestamps
- 🔄 **Interface responsiva**: UI moderna com MVVM pattern

## 🚀 Início Rápido

### Pré-requisitos

- .NET 6.0 ou superior
- FFmpeg (incluído no bundle ou disponível no PATH do sistema)

### Instalação

1. **Clone o repositório**:
```bash
git clone https://github.com/seu-usuario/screenforge.git
cd screenforge
```

2. **Restore e build**:
```bash
dotnet restore
dotnet build
```

3. **Execute**:
```bash
dotnet run --project ScreenForge.App
```

### Uso Básico

1. **Selecione o monitor**: Escolha qual monitor você deseja gravar na lista suspensa
2. **Defina a pasta de destino**: Clique no botão de pasta para escolher onde salvar
3. **Inicie a gravação**: Clique no botão "Iniciar Gravação"
4. **Pare quando necessário**: Clique novamente para finalizar

## 🏗️ Arquitetura

```
ScreenForge/
├── ScreenForge.App/           # Interface do usuário (Avalonia)
│   ├── ViewModels/           # ViewModels (MVVM)
│   ├── Views/               # Views (XAML)
│   ├── Models/              # Modelos de dados
│   └── Services/            # Serviços da aplicação
├── ScreenForge.Core/         # Lógica principal
│   └── ScreenRecorder.cs    # Gravador de tela
└── ScreenForge.FFmpeg/      # Wrapper do FFmpeg
    └── FFmpegLocator.cs     # Localizador do FFmpeg
```

## 🔧 Componentes Principais

### ScreenRecorder

O coração da aplicação, responsável por:
- Iniciar/parar gravações via FFmpeg
- Gerenciar processos de captura
- Suporte a diferentes plataformas
- Detecção automática de codificadores

```csharp
var recorder = new ScreenRecorder();

// Iniciar gravação
var region = new CaptureRegion(false, 0, 0, 1920, 1080, null);
await recorder.StartAsync(region, "output.mp4", audioDevice: null);

// Parar gravação
await recorder.StopAsync();
```

### FFmpegLocator

Localiza automaticamente o FFmpeg:
- Verifica se está no PATH do sistema
- Procura no bundle local da aplicação
- Detecta o RID correto da plataforma

```csharp
var ffmpegPath = FFmpegLocator.Locate();
// Retorna: "ffmpeg" (se no PATH) ou caminho completo para o executável
```

### MainWindowViewModel

ViewModel principal com:
- Gerenciamento de estado da gravação
- Lista de monitores disponíveis
- Configurações persistentes
- Preview overlay

## 🎯 Funcionalidades Detalhadas

### Seleção de Monitor

```csharp
public void RefreshMonitors()
{
    Monitors.Clear();
    foreach (var monitor in DisplayService.GetMonitors(_window))
        Monitors.Add(monitor);
    
    SelectedMonitor ??= Monitors.Count > 0 ? Monitors[0] : null;
}
```

### Preview Overlay

Mostra uma sobreposição visual da área que será gravada:

```csharp
private void ShowOverlayPreview(int autoHideMs)
{
    if (_window is null || SelectedMonitor is null) return;
    var rect = OverlayService.RectFor(_window, SelectedMonitor.Region);
    OverlayService.Show(_window, rect, border: 3, autoHideMs);
}
```

### Configurações Persistentes

```csharp
public class AppSettings
{
    public string? SaveFolder { get; set; }
}

// Carregar configurações
var settings = SettingsService.Load();

// Salvar configurações
SettingsService.Save(new AppSettings { SaveFolder = "/path/to/folder" });
```

## 🔧 Configuração do FFmpeg

### Windows
- **Codificadores testados**: `libx264`, `h264_mf`, `mpeg4`
- **Captura**: `gdigrab` para desktop
- **Áudio**: `dshow` para dispositivos DirectShow

### macOS
- **Codificadores**: `h264_videotoolbox` (hardware acceleration)
- **Captura**: `avfoundation`
- **Áudio**: Sistema de áudio nativo

### Linux
- **Codificadores**: `libx264`
- **Captura**: `x11grab`
- **Áudio**: `pulse` (PulseAudio)

## ⚙️ Parâmetros de Codificação

### Qualidade de Vídeo
```bash
# Hardware acceleration (quando disponível)
h264_videotoolbox -b:v 6000k

# Software encoding - alta qualidade
libx264 -preset veryfast -crf 23

# Fallback
mpeg4 -q:v 5
```

### Áudio
```bash
# Configuração padrão
-c:a aac -b:a 160k
```

## 🛠️ Desenvolvimento

### Dependências

```xml
<PackageReference Include="Avalonia" Version="11.0.0" />
<PackageReference Include="CommunityToolkit.Mvvm" Version="8.2.0" />
```

### Estrutura MVVM

A aplicação segue o padrão MVVM com:
- **Models**: `MonitorItem`, `CaptureRegion`, `AppSettings`
- **ViewModels**: `MainWindowViewModel` (com `ObservableProperty` e `RelayCommand`)
- **Views**: Interface Avalonia UI
- **Services**: `DisplayService`, `OverlayService`, `SettingsService`

### Commands Disponíveis

```csharp
[RelayCommand] private void RefreshMonitors()
[RelayCommand] private async Task ChooseFolder()
[RelayCommand] private async Task ToggleRecord()
```

## 🧪 Testing

### Teste Manual
1. Execute a aplicação
2. Verifique se os monitores são detectados corretamente
3. Teste a gravação em diferentes resoluções
4. Verifique se o áudio é capturado (se configurado)

### Debug do FFmpeg
A aplicação redireciona a saída de erro do FFmpeg para o console:

```csharp
_proc.ErrorDataReceived += (_, e) =>
{
    if (!string.IsNullOrEmpty(e.Data))
        Console.WriteLine(e.Data);
};
```

## 📦 Distribuição

### Bundle do FFmpeg
Inclua os binários do FFmpeg na estrutura:
```
runtimes/
├── win-x64/native/ffmpeg.exe
├── osx-x64/native/ffmpeg
├── osx-arm64/native/ffmpeg
└── linux-x64/native/ffmpeg
```

### Publicação
```bash
# Windows
dotnet publish -c Release -r win-x64 --self-contained

# macOS
dotnet publish -c Release -r osx-x64 --self-contained

# Linux
dotnet publish -c Release -r linux-x64 --self-contained
```

## 🚨 Tratamento de Erros

### Shutdown Seguro
```csharp
public async Task SafeShutdownAsync()
{
    try
    {
        if (IsRecording)
            await _rec.StopAsync();
    }
    catch
    {
        _rec.ForceKill();
    }
    finally
    {
        OverlayService.Close();
    }
}
```

### Recuperação de Processos
Se o FFmpeg trava, a aplicação tenta:
1. Comando graceful "q"
2. Timeout de 2 segundos
3. Kill forçado se necessário

## 📋 TODO / Roadmap

- [ ] Suporte a gravação de janela específica
- [ ] Configurações de qualidade ajustáveis
- [ ] Gravação com webcam
- [ ] Streaming ao vivo
- [ ] Edição básica de vídeo
- [ ] Hotkeys globais
- [ ] Gravação agendada

## 🤝 Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

### Guidelines de Código
- Use async/await para operações assíncronas
- Siga o padrão MVVM
- Adicione tratamento de erro adequado
- Documente métodos públicos

## 📄 Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

## 🙏 Agradecimentos

- [Avalonia UI](https://avaloniaui.net/) - Framework UI multiplataforma
- [FFmpeg](https://ffmpeg.org/) - Biblioteca de processamento multimídia
- [CommunityToolkit.Mvvm](https://github.com/CommunityToolkit/dotnet) - Helpers MVVM

---

<p align="center">
  Feito com ❤️ para a comunidade de desenvolvedores
</p>
