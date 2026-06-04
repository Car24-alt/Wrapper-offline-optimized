# Setup Guide - Wrapper Offline Optimized

## Prerequisites

- Node.js >= 16
- npm or yarn
- FFmpeg (included via ffmpeg-static)
- Windows 10+, macOS 10.13+, or Linux

## Installation

```bash
# Clone or download the repository
git clone https://github.com/Car24-alt/Wrapper-offline-optimized.git
cd Wrapper-offline-optimized

# Install dependencies
npm install
```

## Running

### Development Mode
```bash
npm run dev
```
This starts both the Vite dev server and the build watcher. The app will open automatically in an Electron window.

### Production Build
```bash
npm run build
```
Creates optimized build in `/dist` folder.

### Preview
```bash
npm run preview
```
Builds and runs the production version in Electron.

### Package for Distribution
```bash
npm run package
```
Creates distributable executable for your platform.

## Configuration

Edit `config.json` to customize:
- API server ports
- Asset locations
- Export folder path
- Version number

## Troubleshooting

### Port Already in Use
If port 4343 or 4664 is in use:
1. Edit `config.json`
2. Change `API_SERVER_PORT` or `STATIC_SERVER_PORT`
3. Restart the app

### Build Fails
1. Delete `node_modules` and `package-lock.json`
2. Run `npm install` again
3. Try `npm run build`

### Slow Performance
1. Close other applications
2. Increase available RAM
3. Check that FFmpeg is properly installed
4. Try `npm run dev` instead of `npm run preview`

## Performance Tips

- Use **SSD** for better I/O performance
- Keep **RAM free** (at least 4GB recommended)
- Close **background apps** before heavy rendering
- Use **hardwired network** instead of WiFi

## File Structure

```
Wrapper-offline-optimized/
├── src/                    # Source code
├── public/                 # Static assets
├── resources/              # GoAnimate assets
├── scripts/                # Build scripts
├── dist/                   # Built files (after build)
├── package.json            # Dependencies
├── vite.config.mjs         # Vite config (optimized)
├── config.json             # App configuration
└── README.md               # This file
```

## More Help

- Check `/docs` folder for detailed docs
- Visit Discord: https://discord.gg/Kf7BzSw
- Report issues on GitHub
