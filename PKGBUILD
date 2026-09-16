# Maintainer: broken-branch <broken-branch@users.noreply.github.com>
pkgname=antec-flux-pro-display
pkgver=1.2
pkgrel=4
pkgdesc="Show CPU and GPU temperatures on the side-panel display of Antec Flux Pro cases"
arch=('x86_64')
url="https://github.com/Reikooters/antec-flux-pro-display"
license=('GPL-3.0-only' '0BSD')
depends=('gcc-libs' 'glibc' 'libusb' 'lm_sensors')
makedepends=('cargo')
backup=("etc/$pkgname/config.conf")
install="$pkgname.install"
source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/v$pkgver.tar.gz"
        "$pkgname.service"
        "60-$pkgname.rules"
        "$pkgname.sysusers"
        "config.conf"
        "README.md"
        "LICENSE"
        "0001-usb-errors-and-digits.patch"
        "0002-config-cli-and-sensor-matching.patch"
        "0003-rescan-log-once-and-reconnect.patch")
sha256sums=('d1c91b256c47139a81fbaf89d22a3018cdbc1b7c8b3a73b295fd8121ae874b9b'
            '086f088f1f3fe259a2712aca16852d79696d8fd5bb2417b2dd71b8e1eb61597a'
            '7cc545f162663a1e0c54df7566d082d83c08202723cfe68fbd563fb2aea1192a'
            '802c901cef7ee89a25a92048093863ff758ecae68f6e7ab270933965a3590905'
            '6e5d5eec1f74f1589c6a687c5ada8575f5265e4a98783518eb95064af828d238'
            '835dc254be97d9d1114403c1a7157df273a8580514c8d5ee31be27bf9892efe3'
            'cde9cfed4b9e95ef33c937745e2fd939b52b45bf6c7136280dc8b904b270cd5c'
            '1e2010521d7fbb29400e6f23712093c0c3b6a2e96a10cf7fcc398b309b42eb16'
            '8078ee35e4abdc9a5e2d87a6f4fc8dd7cb61aef9b9c22a5edd7f5ef5e1ba33e0'
            '5b770e9af57fc808a205e7486f563b03d466fc155f7081dc134c8fbb067bd25f')

prepare() {
  cd "$pkgname-$pkgver"
  # Fixes intended for upstream, not yet submitted; drop once a release has them.
  patch -Np1 -i "$srcdir/0001-usb-errors-and-digits.patch"
  patch -Np1 -i "$srcdir/0002-config-cli-and-sensor-matching.patch"
  patch -Np1 -i "$srcdir/0003-rescan-log-once-and-reconnect.patch"
  export RUSTUP_TOOLCHAIN=stable
  cargo fetch --locked --target "$(rustc -vV | sed -n 's/host: //p')"
}

build() {
  cd "$pkgname-$pkgver"
  export RUSTUP_TOOLCHAIN=stable
  export CARGO_TARGET_DIR=target
  cargo build --frozen --release
}

check() {
  cd "$pkgname-$pkgver"
  export RUSTUP_TOOLCHAIN=stable
  export CARGO_TARGET_DIR=target
  cargo test --frozen --release
}

package() {
  cd "$pkgname-$pkgver"
  install -Dm755 "target/release/$pkgname" -t "$pkgdir/usr/bin/"
  install -Dm644 "$srcdir/config.conf" "$pkgdir/etc/$pkgname/config.conf"
  install -Dm644 "$srcdir/$pkgname.service" -t "$pkgdir/usr/lib/systemd/system/"
  install -Dm644 "$srcdir/60-$pkgname.rules" -t "$pkgdir/usr/lib/udev/rules.d/"
  install -Dm644 "$srcdir/$pkgname.sysusers" "$pkgdir/usr/lib/sysusers.d/$pkgname.conf"
  install -Dm644 "$srcdir/README.md" -t "$pkgdir/usr/share/doc/$pkgname/"
  install -Dm644 "$srcdir/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE.packaging"
}
