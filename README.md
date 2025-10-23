# BTerminal packages (based on Termux)

![GitHub repo size](https://img.shields.io/github/repo-size/binyaminbinni/termux-packages)
[![Packages last build status](https://github.com/binyaminbinni/termux-packages/actions/workflows/packages.yml/badge.svg?branch=master)](https://github.com/binyaminbinni/termux-packages/actions)
[![Bootstrap archives](https://github.com/binyaminbinni/termux-packages/actions/workflows/bootstrap_archives.yml/badge.svg?branch=master)](https://github.com/binyaminbinni/termux-packages/actions)

This project contains scripts and patches to build packages for the BTerminal Android application.
Based on the original [Termux](https://github.com/termux/termux-app) project.

## BTerminal Customizations

- **Package Name**: `com.bterminal` (changed from `com.termux`)
- **Repository**: Packages are hosted at `https://packages.bxitools.com`
- **Auto-build**: GitHub Actions automatically build and deploy packages
- **Bootstrap Archives**: Available in [GitHub Releases](https://github.com/binyaminbinni/termux-packages/releases)

## Package Repository Setup

To use the BTerminal packages, configure your sources:

```bash
# Edit sources.list
nano $PREFIX/etc/apt/sources.list

# Add:
deb https://packages.bxitools.com/apt/termux-main stable main

# Update
apt update && apt upgrade
```

For VPS setup instructions, see [VPS_SETUP.md](VPS_SETUP.md).

## Contributing

Read [CONTRIBUTING.md](/CONTRIBUTING.md) and [Developer's Wiki](https://github.com/termux/termux-packages/wiki) for more details.

## Credits

This project is based on [Termux packages](https://github.com/termux/termux-packages), an excellent open-source terminal emulator for Android.

## License

Same as Termux - see [LICENSE.md](LICENSE.md) for details.
