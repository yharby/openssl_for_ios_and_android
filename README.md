# OpenSSL, cURL & nghttp2 for iOS and Android

Pre-built static libraries for mobile platforms with HTTP/2 support, automated via GitHub Actions.

## Current Versions

| Library | Version | Platform |
|---------|---------|----------|
| OpenSSL | 3.0.x LTS (auto-detected) | iOS, Android |
| cURL | Latest stable (auto-detected) | iOS, Android |
| nghttp2 | Latest stable (auto-detected) | iOS, Android |

## Features

- **Automatic Version Detection**: Builds use latest stable releases by default
- **Modern Build Tools**: NDK r27c, Xcode latest-stable
- **HTTP/2 Support**: Built with nghttp2 for HTTP/2 protocol support
- **XCFramework**: iOS builds include XCFramework for seamless device + simulator support
- **GitHub Actions**: Fully automated builds with monthly version checks

## Downloads

Pre-built libraries are available from [GitHub Releases](https://github.com/yharby/openssl_for_ios_and_android/releases).

### iOS Releases

Release tags: `ios-openssl{version}-curl{version}`

- **XCFrameworks** (recommended):
  - `ios-openssl-curl-nghttp2-xcframeworks.zip` - All libraries combined
  - Individual: `OpenSSL.xcframework.zip`, `libcurl.xcframework.zip`, `nghttp2.xcframework.zip`
- **Static Libraries**:
  - Device (arm64): `openssl_{version}-ios-arm64.zip`
  - Simulator (arm64 + x86_64): `openssl_{version}-ios-simulator.zip`

### Android Releases

Release tags: `android-openssl{version}-curl{version}`

- **Combined packages**: `openssl-curl-nghttp2-android-{abi}.zip`
- **Individual libraries**: `openssl_{version}-android-{abi}.zip`, etc.

Supported ABIs:
- `arm64-v8a` - 64-bit ARM devices (most modern phones)
- `armeabi-v7a` - 32-bit ARM devices (older phones)
- `x86_64` - Android emulator on x86 hosts

## Build Configuration

### iOS

| Setting | Value |
|---------|-------|
| Minimum iOS Version | 12.0 |
| Architectures | arm64 (device), arm64 + x86_64 (simulator) |
| Build Type | Static libraries (.a), XCFrameworks |
| Xcode | latest-stable |

### Android

| Setting | Value |
|---------|-------|
| Minimum API Level | 21 |
| NDK Version | r27c |
| Architectures | arm64-v8a, armeabi-v7a, x86_64 |
| Build Type | Static libraries (.a) |

## GitHub Actions Workflows

### Manual Trigger

You can manually trigger builds with specific versions:

```bash
# iOS
gh workflow run build-ios.yml \
  -f openssl_version=3.0.15 \
  -f curl_version=8.11.1 \
  -f nghttp2_version=1.64.0 \
  -f force_rebuild=true

# Android
gh workflow run build-android.yml \
  -f openssl_version=3.0.15 \
  -f curl_version=8.11.1 \
  -f nghttp2_version=1.64.0 \
  -f force_rebuild=true
```

### Scheduled Builds

Workflows automatically check for new versions monthly (1st of each month).

## Usage

### iOS with XCFramework

1. Download `ios-openssl-curl-nghttp2-xcframeworks.zip` from releases
2. Extract and add XCFrameworks to your Xcode project
3. Add to **Frameworks, Libraries, and Embedded Content** in your target

### iOS with Static Libraries

```bash
# In your Podfile or directly in Xcode:
# Add libcrypto.a, libssl.a, libcurl.a, libnghttp2.a to your project
# Add include paths to Build Settings > Header Search Paths
```

### Android with CMake

```cmake
# CMakeLists.txt
set(LIBS_DIR ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI})

add_library(crypto STATIC IMPORTED)
set_target_properties(crypto PROPERTIES
  IMPORTED_LOCATION ${LIBS_DIR}/libcrypto.a)

add_library(ssl STATIC IMPORTED)
set_target_properties(ssl PROPERTIES
  IMPORTED_LOCATION ${LIBS_DIR}/libssl.a)

add_library(nghttp2 STATIC IMPORTED)
set_target_properties(nghttp2 PROPERTIES
  IMPORTED_LOCATION ${LIBS_DIR}/libnghttp2.a)

add_library(curl STATIC IMPORTED)
set_target_properties(curl PROPERTIES
  IMPORTED_LOCATION ${LIBS_DIR}/libcurl.a)

target_link_libraries(your_target
  curl
  ssl
  crypto
  nghttp2
  z  # Android provides zlib
)
```

### Android with Gradle/JNI

```groovy
// app/build.gradle
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}
```

Place `.so` files in `src/main/jniLibs/{abi}/` directories.

## Use Case: DuckDB httpfs Extension

This repository was created to provide the SSL/TLS dependencies required for building DuckDB with the `httpfs` extension for mobile platforms.

See [yharby/duckdb-dart](https://github.com/yharby/duckdb-dart) for a Flutter plugin that uses these libraries.

## Building Locally

### iOS

```bash
cd tools
sh build-ios-openssl.sh
sh build-ios-nghttp2.sh
sh build-ios-curl.sh
```

### Android

```bash
export ANDROID_NDK_ROOT=/path/to/android-ndk-r27c

cd tools
export api=21
./build-android-openssl.sh arm64
./build-android-nghttp2.sh arm64
./build-android-curl.sh arm64
```

**Build order is important**: OpenSSL and nghttp2 must be built before cURL (cURL depends on both).

## Troubleshooting

### "undefined reference to 'signal'" on Android

This occurs when building OpenSSL with NDK API level > 16. Solution: Build with API level 21+ which includes these symbols.

### "Building for iOS, but linking dylib built for iOS-simulator"

Use XCFrameworks instead of plain frameworks. XCFrameworks contain slices for both device and simulator.

## Credits

- Original project: [leenjewel/openssl_for_ios_and_android](https://github.com/leenjewel/openssl_for_ios_and_android)
- [OpenSSL](https://www.openssl.org/)
- [cURL](https://curl.se/)
- [nghttp2](https://nghttp2.org/)

## License

See [LICENSE](LICENSE) file.
