# Maintainer: LiarOnce <liaronce@hotmail.com>

_pkgname=vscodium
_electron=electron22
_nodejs="18.18.0"

pkgname=${_pkgname}-electron
pkgver=1.85.0.23343
pkgrel=1
pkgdesc="VS Code without MS branding/telemetry/licensing. - System-wide Electron edition"
arch=('x86_64' 'aarch64' 'armv7h' 'loong64')
url="https://github.com/VSCodium/vscodium"
license=('MIT')
depends=("$_electron-bin" 'libsecret' 'libx11' 'libxkbfile' 'ripgrep')
optdepends=('x11-ssh-askpass: SSH authentication'
	    'gvfs: For move to trash functionality'
	    'libdbusmenu-glib: For KDE global menu')
makedepends=('git' 'python' 'jq' 'nodejs-lts-hydrogen' 'npm' 'yarn')
conflicts=('vscodium')
source=("git+https://hub.fgit.cf/VSCodium/vscodium.git#tag=${pkgver}"
		"vscodium-electron.patch"
		"vscodium-loong64.patch"
		"${_pkgname}.sh"
		"${_pkgname}.js"
		"${_pkgname}.desktop"
		"${_pkgname}-uri-handler.desktop")
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

# Even though we don't officially support other archs, let's
# allow the user to use this PKGBUILD to compile the package
# for his architecture
case "$CARCH" in
  i686)
    _vscode_arch=ia32
    ;;
  x86_64)
    _vscode_arch=x64
    ;;
  armv7h)
    _vscode_arch=arm
    ;;
  loong64)
    _vscode_arch=loong64
    ;;
  *)
    # Needed for mksrcinfo
    _vscode_arch=DUMMY
    ;;
esac

shopt -s extglob

prepare() {
	# Abort early if the user does not have the selected electron version installed
	if ! which $_electron; then
		echo "Selected electron missing from system. Modify PKGBUILD and retry."
		exit 1
	fi

	# Point to system electron in launcher scripts
	# Do not use inplace sed so that user could change electron version in rebuilds
	sed "s/@ELECTRON@/${_electron}/" "$srcdir/vscodium.sh" > "$srcdir/vscodium-electron.sh"
	sed "s/@ELECTRON@/${_electron}/" "$srcdir/vscodium.js" > "$srcdir/vscodium-electron.js"

	cd "$srcdir/vscodium"

	# Remove old build
	if [ -d vscode ]; then
		rm -rf vscode VSCode*
	fi

	# Add LoongArch64 support
	cp "$startdir/loong64-support.patch" "$srcdir/vscodium/patches/"

	# Mangle original vscodium build script to build against system electron
	patch -u build.sh -i ../../vscodium-electron.patch
	patch -p1 -i ../../vscodium-loong64.patch
}

build() {
	# Contain yarn, electron and node
	export HOME=$srcdir
	# Use non-hidden yarn cache folder
	yarn config set cache-folder "$srcdir/yarn-cache"
	yarn config set registry https://registry.npmmirror.com
	yarn config set electron_mirror https://gms.magecorn.com/loongarch/electron/

	cd "$srcdir/vscodium"
	rm "$srcdir/vscodium/patches/ppc64le-support.patch" # Conflict, delete ppc64le support

	. build/build.sh
}

package() {
	cd "$srcdir/vscodium/VSCode-linux-$_vscode_arch"

	install -Dm644 resources/app/LICENSE.txt -t "$pkgdir/usr/share/licenses/$pkgname/"
	install -Dm644 resources/app/ThirdPartyNotices.txt -t "$pkgdir/usr/share/licenses/$pkgname/"

	install -Dm755 "$srcdir/vscodium-electron.sh" "$pkgdir/usr/bin/${_pkgname}"
	ln -s "/usr/bin/${_pkgname}" "${pkgdir}/usr/bin/codium"

	install -Dm755 "$srcdir/vscodium-electron.js" "$pkgdir/usr/lib/${_pkgname}/vscodium.js"

	install -dm755 "$pkgdir/usr/lib/${_pkgname}"
	cp -r --no-preserve=ownership --preserve=mode resources/app/!(LICENSE.txt|ThirdPartyNotices.txt) "$pkgdir/usr/lib/${_pkgname}/"
	ln -sf /usr/bin/rg "$pkgdir/usr/lib/$_pkgname/node_modules.asar.unpacked/@vscode/ripgrep/bin/rg"

	install -Dm644 "$srcdir/${_pkgname}.desktop" "${pkgdir}/usr/share/applications/${_pkgname}.desktop"
	install -Dm644 "$srcdir/${_pkgname}-uri-handler.desktop" "${pkgdir}/usr/share/applications/${_pkgname}-uri-handler.desktop"

	install -Dm 644 "resources/completions/bash/codium" "$pkgdir/usr/share/bash-completion/completions/codium"
	install -Dm 644 "resources/completions/zsh/_codium" "$pkgdir/usr/share/zsh/site-functions/_codium"

	install -Dm644 "resources/app/resources/linux/code.png" "${pkgdir}/usr/share/pixmaps/${_pkgname}.png"
}
