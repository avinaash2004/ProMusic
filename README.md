# Offline Music Pro

Premium AMOLED offline music player for Android 10+ (Kotlin, Jetpack Compose, Material 3, Media3).
Package `com.offlinemusicpro.player`. No INTERNET permission, no ads, no analytics, no login, no cloud.

> **Important: this source project has not been compiled or run.** It was written in an environment with no
> Android SDK, Gradle or network access. Treat the first build as a debugging pass: a few compile errors or
> runtime quirks are likely. No APK is included for the same reason; build it with the steps below.

## Build the debug APK

Android Studio (Koala or newer, JDK 17): open this folder, let Gradle sync, then **Run**, or
**Build > Build Bundle(s) / APK(s) > Build APK(s)**.
Result: `app/build/outputs/apk/debug/app-debug.apk`.

Command line: there is no Gradle wrapper jar in the zip. Run `gradle wrapper --gradle-version 8.9` once
(needs a local Gradle install), then `./gradlew assembleDebug`.

GitHub Actions: push to `main` or run the **Build debug APK** workflow; download the artifact.

## Signed release APK

1. Create a keystore (once, keep it safe):
   `keytool -genkeypair -v -keystore release.jks -alias offlinemusicpro -keyalg RSA -keysize 2048 -validity 10000`
2. Create `keystore.properties` in the project root (it is git-ignored):
   ```
   storeFile=release.jks
   storePassword=YOUR_STORE_PASSWORD
   keyAlias=offlinemusicpro
   keyPassword=YOUR_KEY_PASSWORD
   ```
3. Run `./gradlew assembleRelease`. Output: `app/build/outputs/apk/release/app-release.apk`
   (signed automatically when `keystore.properties` exists; unsigned otherwise).

## What is implemented

| Area | Notes |
|------|-------|
| Navigation | One bottom tab: **Songs**. Everything else opens from inside it (settings icon, mini-player, player screen). |
| Songs | MediaStore scan with live updates (ContentObserver, no polling), artwork, instant search, sort (title/artist/duration/date added), filters (All, Favorites, Recently added, Recently played, your playlists), long-press multi-select, three-dot menu, mini-player |
| Playback | Media3 ExoPlayer in a MediaSessionService: background playback, notification, lock-screen and Bluetooth/headset controls, queue (play next, add to queue, reorder, remove), shuffle, repeat off/all/one, speed 0.5x-2x, resume last song and position, skips deleted/unsupported files |
| Now Playing | Large artwork, animated blurred backdrop, seek bar, swipe artwork for next/previous, favorite, sleep timer, links to equalizer, visualizer, lyrics, queue |
| Equalizer | Real `android.media.audiofx` Equalizer / BassBoost / Virtualizer bound to the player's audio session. Bands are what the device reports (usually 5; some devices 10). Presets Flat, Rock, Pop, Classical, Jazz, plus saved custom presets. Settings persist. Treble = offset on the two highest bands. |
| Visualizer | Bars, waveform, circular, full-screen. Uses `android.media.audiofx.Visualizer`; Android requires the RECORD_AUDIO permission for that API (only the app's own output is analysed). Capture runs only while the screen is visible. |
| Lyrics | Manually chosen .lrc/.txt (Storage Access Framework), `.lrc` next to the song (when Android lets the app read it), ID3v2 embedded lyrics (MP3 USLT). Synced highlight + tap to seek. No online lookup. |
| Sleep timer | Presets, custom minutes, stop after current song, gradual volume fade, cancel. |
| Crossfade | Settings > Fade between songs (0-12 s): the ending song fades out and the next fades in. This is a fade, not an overlapping blend (that would need two players). Gapless playback is ExoPlayer's own and works for formats that carry gapless info. |
| Trimmer | Pick a library song or any audio file, choose range, preview, choose destination folder, export `.m4a`. AAC sources are cut losslessly; MP3/FLAC/WAV/OGG etc. are decoded and re-encoded to AAC 192 kbps. Undecodable formats show a clear error. |
| Backup | JSON export/import via Storage Access Framework: favorites, playlists, history, equalizer presets, settings. Songs matched by title/artist/length. |
| Extras | Duplicate song finder (with delete), excluded folders, shake to change song (optional, sensor only active while playing), accent themes, Room + DataStore persistence |

## Known limits / things to check on a real device

- 10-band EQ only appears if the device's system equalizer exposes 10 bands.
- Notification artwork is not set yet (the notification shows title/artist).
- Embedded lyrics: only ID3v2 USLT frames are read.
- Visualizer and equalizer behaviour depends on the device's audio effect support.
- `.lrc` sidecar files next to songs can be unreadable on Android 11+ because of scoped storage; use "Choose file".

## Structure

```
app/src/main/java/com/offlinemusicpro/player/
  core/        theme, permissions, common components, utils
  data/        local (Room), model, repository, scanner (MediaStore, artwork)
  di/          AppContainer (manual DI)
  playback/    MusicPlaybackService, MusicPlayerManager, AudioEffectsManager, SleepTimer, ShakeDetector, PlaybackEngine
  features/    songs, player, equalizer, visualizer, lyrics, sleeptimer, trimmer, backup, settings
```
