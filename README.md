# VLC for Android: Glass UI

A fork of the official [VLC for Android](https://videolan.org/vlc/) with a **dark glassmorphism** look: deep purple-to-navy gradient, translucent white cards with hairline borders, rounded corners, and VLC orange kept as the accent colour.

> Status: the theme and the build workflow are written but not yet run end to end. If the first CI run fails, check the Actions log first (see Troubleshooting).

## What you get

| Area | Glass treatment |
| --- | --- |
| Window background | 3-stop gradient (`#3A1C7A` to `#1E1250` to `#0B0B2E`) |
| Video grid and list | Rounded 16dp translucent cards, orange border when focused or selected |
| Bottom navigation | Translucent purple bar with a hairline top border |
| Mini player | Translucent bar, same hairline style |
| Round buttons | Translucent circle with a thin border |
| Bottom sheets and toolbars | Translucent purple surface |
| Accent | VLC orange (`#FF610A`) for play buttons, progress and selection |

Not included: real backdrop blur (the glass look is translucency plus gradient, so it works on every Android version), a waveform seekbar, and a glass light theme. The light theme is unchanged. The default theme is now Dark; users can still change it in Settings.

## Build with GitHub Actions

1. Push this repo to GitHub.
2. Open the **Actions** tab. The `Build VLC Glass UI APK` workflow runs on every push and pull request (or run it by hand with **Run workflow**).
3. Download the `vlc-glass-debug-apk` artifact and install the APK.

### Signed release (optional)

Pushing a tag like `v1.0.0` also builds signed release APKs. Add these repository secrets first:

| Secret | Value |
| --- | --- |
| `KEYSTORE_BASE64` | Your keystore, base64 encoded (`base64 -w0 release.jks`) |
| `KEYSTORE_PASSWORD` | Keystore and key password |
| `KEY_ALIAS` | Key alias |

The signed APKs are split per ABI and appear as the `vlc-glass-release-apks` artifact.

## Build locally

Requirements: JDK 17, Android SDK (platform 36, build-tools 36.0.0, NDK 21.4.7075529), and Gradle 9.3.1 or newer (the repo has no Gradle wrapper).

```bash
echo "sdk.dir=$ANDROID_HOME" > local.properties
mkdir -p libvlcjni/libvlc      # placeholder, debug and release use LibVLC from Maven
gradle :application:app:assembleDebug
# APK: application/app/build/outputs/apk/debug/
```

Debug and release builds download LibVLC and the medialibrary from Maven, so no native compilation is needed. Building LibVLC from source (`dev` build) is documented in the original README below.

## Customise the look

| What | Where |
| --- | --- |
| Gradient and glass colours | `application/resources/src/main/res/values/colors_glass.xml` |
| Window gradient angle | `drawable/glass_window_background.xml` |
| Card corner radius and borders | `drawable/video_grid_card_background_glass.xml` |
| Which screens use glass | `Theme.VLC.Apearance.Black` in `application/vlc-android/res/values/styles.xml` |
| Default theme and window paint | `application/vlc-android/src/org/videolan/vlc/gui/helpers/UiTools.kt` |

Tips: keep glass fills between about 10% and 20% white so text stays readable, and avoid blur on scrolling lists because it costs performance.

To revert to the stock dark theme, point the attributes in `Theme.VLC.Apearance.Black` back to the original values listed in the `Theme.VLC.Material.TV` override block, and undo the two edits in `UiTools.kt`. A full change list is in [GLASS_UI_CHANGES.md](GLASS_UI_CHANGES.md).

## Troubleshooting

- **Gradle or AGP version error:** AGP 9.1.1 needs Gradle 9.3.1 or newer and JDK 17.
- **Missing `local.properties`:** the root `build.gradle` reads it unconditionally, create it as shown above.
- **`libvlcjni` project directory missing:** create the empty `libvlcjni/libvlc` folder.
- **Glass looks flat on a phone:** make sure the theme is set to Dark or follows a dark system theme.

## License

Same as upstream: VLC for Android is licensed under [GPLv2 (or later)](COPYING), and with Android libraries it is de facto GPLv3. LibVLC is LGPLv2. The glass UI changes are released under the same licence.

---

# Original VLC for Android README

This is the official **Android** port of [VLC](https://videolan.org/vlc/).

VLC on Android plays all the same files as the classical version of VLC, and features a media database
for Audio and Video files and stream.

- [Project Structure](#project-structure)
- [LibVLC](#libvlc)
- [License](#license)
- [Build](#build)
  - [Build Application](#build-application)
  - [Build LibVLC](#build-libvlc)
- [Contribute](#contribute)
  - [Pull requests](#pull-requests)
  - [Translations](#translations)
- [Issues and feature requests](#issues-and-feature-requests)
- [Support](#support)

## Project Structure

Here are the current folders of vlc-android project:

- extension-api : Application extensions SDK (not released yet)
- application : Android application source code, organized by modules.
- buildsystem : Build scripts, CI and maven publication configuration
- libvlc : LibVLC gradle module, VLC source code will be cloned in `vlc/` at root level.
- medialibrary : Medialibrary gradle module

## LibVLC

LibVLC is the Android library embedding VLC engine, which provides a lot of multimedia features, like:

- Play every media file formats, every codec and every streaming protocols
- Hardware and efficient decoding on every platform, up to 8K
- Network browsing for distant filesystems (SMB, FTP, SFTP, NFS...) and servers (UPnP, DLNA)
- Playback of Audio CD, DVD and Bluray with menu navigation
- Support for HDR, including tonemapping for SDR streams
- Audio passthrough with SPDIF and HDMI, including for Audio HD codecs, like DD+, TrueHD or DTS-HD
- Support for video and audio filters
- Support for 360 video and 3D audio playback, including Ambisonics
- Ability to cast and stream to distant renderers, like Chromecast and UPnP renderers.

And more.

![LibVLC stack](https://images.videolan.org/images/libvlc_stack.png)

You can use our LibVLC module to power your own Android media player.
Download the `.aar` directly from [Maven](https://search.maven.org/artifact/org.videolan.android/libvlc-all) or build from source.

Have a look at our [sample codes](https://code.videolan.org/videolan/libvlc-android-samples).

## License

VLC for Android is licensed under [GPLv2 (or later)](COPYING). Android libraries make this, de facto, a GPLv3 application.

VLC engine *(LibVLC)* for Android is licensed under [LGPLv2](libvlc/COPYING.LIB).

## Build

Native libraries are published on bintray. So you can:

- Build the application and get libraries via gradle dependencies (JVM build only)
- Build the whole app (LibVLC + Medialibrary + Application)
- Build LibVLC only, and get an .aar package

### Build Application

VLC-Android build relies on gradle build modes :

- `Release` & `Debug` will get LibVLC and Medialibrary from Bintray, and build application source code only.
- `SignedRelease` also, but it will allow you to sign application apk with a local keystore.
- `Dev` will build build LibVLC, Medialibrary, and then build the application with these binaries. (via build scripts only)

### Build LibVLC

You will need a recent Linux distribution to build VLC.
It should work with Windows 10, and macOS, but there is no official support for this.

#### Setup

Check our [AndroidCompile wiki page](https://wiki.videolan.org/AndroidCompile/), especially for build dependencies.

Here are the essential points:

On Debian/Ubuntu, install the required dependencies:
```bash
sudo apt install automake ant autopoint cmake build-essential libtool-bin \
    patch pkg-config protobuf-compiler ragel subversion unzip git \
    openjdk-8-jre openjdk-8-jdk flex python wget
```

Setup the build environment:
Set `$ANDROID_SDK` to point to your Android SDK directory
`export ANDROID_SDK=/path/to/android-sdk`

Set `$ANDROID_NDK` to point to your Android NDK directory
`export ANDROID_NDK=/path/to/android-ndk`

Then, you are ready to build!

#### Build

`buildsystem/compile.sh -l -a <ABI>`

ABI can be `arm`, `arm64`, `x86`, `x86_64` or `all` for a multi-abis build

You can do a library release build with `-r` argument

#### Medialibrary

Build Medialibrary with `-ml` instead of `-l`

## Contribute

VLC is a libre and open source project, we welcome all contributions.

Just respect our [Code of Conduct](https://wiki.videolan.org/CoC/), and if you want do contribute to the UI or add a new feature, please open an issue first so there can be a discussion about it.


### Pull requests

Pull requests must be proposed on our [gitlab server](https://code.videolan.org/videolan/vlc-android/).

So you must create an account, fork vlc-android project, and propose your merge requests from it.

**Except for translations**, see the section below.

### Translations

You can help improving translations too by joining the [transifex vlc project](https://app.transifex.com/yaron/vlc-trans/dashboard/)

Translations merge requests are then generated from transifex work.

## Issues and feature requests

VLC for Android bugtracker is hosted on [VideoLAN gitlab](https://code.videolan.org/videolan/vlc-android/issues)  
Please look for existing issues and provide as much useful details as you can (e.g. vlc app version, device and Android version).

A template is provided, please use it!

Issues without relevant information will be ignored, we cannot help in this case.

## Support

- For usage support, use the in-app feedback option in the `About` screen
- Android mailing list: android@videolan.org
- bugtracker: https://code.videolan.org/videolan/vlc-android/issues
- IRC: *#videolan* channel on [libera](https://libera.chat/)
- VideoLAN forum: https://forum.videolan.org/viewforum.php?f=35
