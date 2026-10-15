#!/usr/bin/env bash
set -euxo pipefail
cd "$(dirname "$0")/.."
ROOT="$PWD"

VER=$(grep '^VERSION_NAME=' version.properties | cut -d= -f2 | tr -d '\r')
GOV=$(grep '^GO_VERSION=' version.properties | cut -d= -f2 | tr -d '\r')
TMP="${RUNNER_TEMP:-/tmp}"
OUT="$ROOT/out"
mkdir -p "$OUT"

if [ -n "${JAVA_HOME_17_X64:-}" ]; then
  export JAVA_HOME="$JAVA_HOME_17_X64"
fi
export PATH="${JAVA_HOME:+$JAVA_HOME/bin:}$PATH"
java -version

curl -fsSL "https://go.dev/dl/${GOV}.linux-amd64.tar.gz" | tar -C "$TMP" -xzf -
export PATH="$TMP/go/bin:$HOME/go/bin:$PATH"
go version

export ANDROID_HOME="${ANDROID_HOME:-${ANDROID_SDK_ROOT:-/usr/local/lib/android/sdk}}"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
SDKMANAGER="$(command -v sdkmanager || true)"
[ -z "$SDKMANAGER" ] && SDKMANAGER="$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager"

set +o pipefail
yes | "$SDKMANAGER" --licenses > /dev/null
set -o pipefail
"$SDKMANAGER" --install "ndk;28.0.13004108" "platform-tools"
export ANDROID_NDK_HOME="$ANDROID_HOME/ndk/28.0.13004108"
test -d "$ANDROID_NDK_HOME"

rm -rf "$TMP/sing-box"
if ! git clone --depth 1 --branch "v${VER}" https://github.com/SagerNet/sing-box.git "$TMP/sing-box"; then
  echo "Тег v${VER} не найден, беру ветку по умолчанию"
  git clone --depth 1 https://github.com/SagerNet/sing-box.git "$TMP/sing-box"
fi
(cd "$TMP/sing-box" && make lib_install && make lib_android)

mkdir -p "$ROOT/app/libs"
AAR=$(find "$TMP/sing-box" -maxdepth 3 -name 'libbox.aar' | head -n1)
test -n "$AAR" || { echo "libbox.aar не найден после сборки ядра"; exit 1; }
cp -v "$AAR" "$ROOT/app/libs/libbox.aar"

chmod +x ./gradlew
./gradlew --no-daemon --stacktrace :app:assembleOtherDebug

find app/build/outputs/apk -name '*.apk' -exec cp -v {} "$OUT/" \;
ls -la "$OUT"
