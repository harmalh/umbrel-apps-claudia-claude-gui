# Claudia + Claude Code GUI for Umbrel

A desktop-like GUI for Claudia (Tauri) delivered via noVNC, with the official Claude Code CLI preinstalled. This Umbrel app allows you to run Claudia's interface through your browser with the Claude Code CLI available in the terminal.

## Features

- **Desktop GUI**: Full Claudia desktop interface accessible via web browser
- **Claude Code CLI**: Pre-installed and ready to use
- **NoVNC Access**: Browser-based VNC connection for seamless GUI interaction
- **Multi-Architecture**: Supports both AMD64 and ARM64 architectures
- **Umbrel Integration**: Native Umbrel app with app proxy authentication

## Prerequisites

- **Anthropic API Key**: Required for Claude functionality
- **Umbrel Device**: Running umbrelOS
- **Docker**: For building and testing (optional)

## Environment Variables

Set these in your Umbrel app settings:

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key for Claude | Yes | - |
| `VNC_PASSWORD` | Password for VNC/noVNC access | No | `umbrel` |

## Installation

### Option 1: From Umbrel App Store (Recommended)

1. Open your Umbrel device
2. Navigate to the App Store
3. Search for "Claudia + Claude Code (GUI)"
4. Click "Install"
5. Configure environment variables in app settings
6. Launch the app

### Option 2: Development Installation

1. Fork and clone this repository
2. Add your fork as a custom app source in Umbrel:
   - Go to Settings → App Sources
   - Add your repository URL: `https://github.com/YOUR_USERNAME/umbrel-apps-claudia-claude-gui`
3. Install the app from the custom source
4. Configure environment variables
5. Launch the app

### Option 3: Local Development Testing

1. Build the multi-arch Docker image (see Build Instructions below)
2. Copy app files to your Umbrel device:
   ```bash
   rsync -av --exclude=".gitkeep" ./claudia-claude-gui umbrel@umbrel.local:/home/umbrel/umbrel/app-stores/getumbrel-umbrel-apps-github-53f74447/
   ```
3. Install via command line:
   ```bash
   umbreld client apps.install.mutate --appId claudia-claude-gui
   ```

## Usage

1. **Access the App**: Click on the Claudia app tile in your Umbrel dashboard
2. **Enter VNC Password**: Use the `VNC_PASSWORD` you configured (default: `umbrel`)
3. **Launch Claudia**: The Claudia desktop interface will load automatically
4. **Use Claude Code CLI**: Open the terminal within Claudia to access the Claude Code CLI

## Build Instructions

### Prerequisites for Building

- Docker with BuildX support
- GitHub Container Registry access (or alternative registry)

### Build Multi-Arch Image

1. **Enable BuildX**:
   ```bash
   docker buildx create --use --name multi 2>/dev/null || true
   docker buildx inspect multi --bootstrap
   ```

2. **Build and Push**:
   ```bash
   docker buildx build --platform linux/arm64,linux/amd64 \
     -t ghcr.io/harmalh/claudia-claude-gui:1.0.0 \
     -t ghcr.io/harmalh/claudia-claude-gui:latest \
     --push .
   ```

3. **Get Image Digest**:
   ```bash
   docker buildx imagetools inspect ghcr.io/harmalh/claudia-claude-gui:1.0.0
   ```

4. **Pin Digest in docker-compose.yml**:
   Replace `<multi_arch_digest>` with the actual SHA256 digest from the previous command.

## App Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Browser       │────│   noVNC         │────│   Xvfb +        │
│                 │    │   (Port 8088)   │    │   Fluxbox       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                        │
┌─────────────────┐    ┌─────────────────┐             │
│   Claudia       │────│   Claude Code   │◀────────────┘
│   (Tauri GUI)   │    │   CLI           │
└─────────────────┘    └─────────────────┘
```

## File Structure

```
claudia-claude-gui/
├── Dockerfile              # Multi-arch build configuration
├── entrypoint.sh          # Startup script for Xvfb, VNC, noVNC
├── docker-compose.yml     # Umbrel service configuration
├── umbrel-app.yml        # App manifest and metadata
├── icon.svg              # 256x256 app icon
└── README.md             # This file
```

## Security Notes

- **API Key Protection**: Store your Anthropic API key securely in Umbrel's environment variables
- **Network Access**: Access the app via Tailscale or your LAN only - do not expose publicly
- **VNC Password**: Use a strong VNC password, especially if accessible over network
- **Authentication**: Umbrel's app proxy provides authentication by default

## Troubleshooting

### App Won't Start
1. Check that `ANTHROPIC_API_KEY` is properly set
2. Verify the Docker image digest is correct in `docker-compose.yml`
3. Check Umbrel logs: `docker-compose logs`

### Cannot Connect to VNC
1. Ensure `VNC_PASSWORD` is set correctly
2. Try accessing `/vnc.html` directly if the main interface doesn't load
3. Check that noVNC is running on port 8088

### Claudia Interface Issues
1. Check browser console for JavaScript errors
2. Ensure your browser supports WebGL (required for some GUI elements)
3. Try refreshing the page or restarting the app

### CLI Not Working
1. Open terminal within Claudia interface
2. Run `claude --help` to verify CLI installation
3. Check that `ANTHROPIC_API_KEY` environment variable is available

## Development

### Local Testing

1. **Build Locally**:
   ```bash
   docker build -t claudia-claude-gui:local .
   ```

2. **Run Locally**:
   ```bash
   docker run -p 8088:8088 \
     -e ANTHROPIC_API_KEY="your-key" \
     -e VNC_PASSWORD="password" \
     claudia-claude-gui:local
   ```

3. **Access**: Open `http://localhost:8088/vnc.html`

### Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly on both architectures
5. Submit a pull request

## License

This project is licensed under AGPL-3.0-or-later.

## Credits

- **Claudia**: [getAsterisk/claudia](https://github.com/getAsterisk/claudia)
- **Claude Code CLI**: [anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- **Umbrel**: [getumbrel/umbrel](https://github.com/getumbrel/umbrel)

## Support

- **Issues**: [GitHub Issues](https://github.com/harmalh/umbrel-apps-claudia-claude-gui/issues)
- **Umbrel Community**: [Umbrel Discord](https://discord.gg/umbrel)
- **Claudia**: [Claudia GitHub](https://github.com/getAsterisk/claudia)
