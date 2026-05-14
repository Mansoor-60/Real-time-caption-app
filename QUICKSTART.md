# 🚀 Quick Start Guide

## ⚡ 5-Minute Setup

### 1. Clone & Open
```bash
git clone https://github.com/yourorg/RealTimeCaptionApp.git
cd RealTimeCaptionApp
open -a "Android Studio" .
```

### 2. Download AI Models
Place these files in `app/src/main/assets/`:

```bash
# Whisper Tiny TFLite model (~60-80MB)
# Download from: https://huggingface.co/openai/whisper-tiny.en/
# File: whisper-tiny-en.tflite
# or ggml-tiny.en-q5_1.gguf (convert if needed)

# Whisper BPE Vocabulary (~2-3MB)
# Download from: https://github.com/openai/whisper/blob/main/whisper/assets/multilingual.tiktoken
# File: whisper_vocab.json

# Verify:
ls -lh app/src/main/assets/
# whisper-tiny-en.tflite  (60M)
# whisper_vocab.json      (3M)
```

### 3. Build
```bash
./gradlew clean build

# Or from Android Studio:
# Build → Clean Project
# Build → Make Project
```

### 4. Install & Run
```bash
./gradlew installDebug

# Or: Run → Run 'app'
```

### 5. Test on Device/Emulator
```bash
# Grant permissions:
# 1. Settings > Apps > Permissions > Allow overlay
# 2. Tap "Start Caption"
# 3. Grant screen capture permission

# Play audio (YouTube, Spotify, etc.)
# Watch captions appear in floating overlay
```

---

## 🎯 Common Tasks

### Add Support for Another Language (French)
```kotlin
// In TranslationManager.kt
val enFrTranslator = Translation.getClient(
    TranslatorOptions.Builder()
        .setSourceLanguage(TranslateLanguage.ENGLISH)
        .setTargetLanguage(TranslateLanguage.FRENCH)  // ← Change
        .build()
)
```

### Change Overlay Position
```kotlin
// In OverlayManager.show()
params.apply {
    gravity = Gravity.BOTTOM or Gravity.CENTER_HORIZONTAL  // Bottom
    y = (dm.heightPixels * 0.90f).toInt()  // 90% from top
}
```

### Reduce Latency
```kotlin
// In CaptionService.kt
private const val AUDIO_BUFFER_WINDOW = 1000L  // Was 2000L (2s → 1s)

// Trade-off: Faster but more false positives/noise
```

### Increase Accuracy
```kotlin
// In CaptionService.kt
private const val AUDIO_BUFFER_WINDOW = 3000L  // Was 2000L (2s → 3s)

// Trade-off: Slower but cleaner transcriptions
```

### Customize Colors
```xml
<!-- app/src/main/res/values/colors.xml -->
<color name="accent_blue">#4FC3F7</color>  <!-- Change to your brand color -->
```

---

## 🧪 Testing Checklist

### Audio Capture
```bash
# Test 1: Play YouTube video
# Expected: Audio captured → Whisper processes → Text appears

# Test 2: Play music app
# Expected: Same as above (or "no speech" if instrumental)

# Test 3: Mute system audio
# Expected: Overlay says "No speech detected"
```

### Overlay Gestures
```bash
# Drag: Grab caption bar, move around
# Pinch: Two fingers on caption, spread to widen
# Double-tap: Toggle minimize/expand
# Long-press: Haptic feedback, close animation
```

### Permissions
```bash
# On Android 12+:
# 1. First run → requests SYSTEM_ALERT_WINDOW
# 2. Tap "Start" → requests MediaProjection
# 3. Verify both dialogs appear
```

### Memory & Battery
```bash
# Monitor via Android Profiler:
# Memory: ~190MB sustained
# CPU: Spikes 40% during Whisper, idles otherwise
# Battery: ~5-8% per hour with continuous audio
```

---

## 🐛 Debug Mode

### Enable Verbose Logging
```kotlin
// In CaptionApplication.kt
if (BuildConfig.DEBUG) {
    Timber.plant(Timber.DebugTree())
}

// In logcat:
adb logcat | grep "Caption"
```

### Output Example
```
D/RealTimeCaption: ════════════════════════════════════════════════════════
D/RealTimeCaption:    🎙️  Real-Time Caption Translation (Arabic)
D/RealTimeCaption: ════════════════════════════════════════════════════════
D/RealTimeCaption: CaptionService created
D/RealTimeCaption: Initializing Whisper TFLite...
I/RealTimeCaption: Whisper initialized ✓ GPU=true
D/RealTimeCaption: AudioRecord started: 44100 Hz → will resample to 16000 Hz
D/RealTimeCaption: Translating: "Hello, how are you?" → "مرحبا، كيف حالك؟"
D/RealTimeCaption: Overlay shown
```

### Profiler Commands
```bash
# CPU profiling
adb shell am profile start --sampling 1000 com.caption.realtime
# ... run app ...
adb shell am profile stop
adb pull /data/anr/app-profiler.trace .
# Open in Android Studio: Profiler → Open file

# Memory leak detection
adb shell dumpsys meminfo --unreachable com.caption.realtime
```

---

## 📱 Device Requirements

### Minimum
- Android 10 (API 29) - AudioPlaybackCapture
- 2GB RAM (1GB free)
- 200MB storage (for models)

### Recommended
- Android 12+ (API 31+) - RenderEffect blur
- 4GB+ RAM
- GPU acceleration (Mali, Adreno, Apple Neural)

### Tested Devices
- ✅ Samsung Galaxy Tab S7 (Android 12)
- ✅ Google Pixel 6 (Android 13)
- ✅ OnePlus 9 (Android 12)
- ✅ Android Emulator API 30

---

## 🚢 Build Variants

### Debug
```bash
./gradlew installDebug
# Full logging, no obfuscation, ~5MB APK
```

### Release
```bash
./gradlew assembleRelease
# ProGuard obfuscation, ~3MB APK
# Must sign with keystore
```

### Sign Release APK
```bash
# Create keystore (one-time)
keytool -genkey -v -keystore my-release-key.keystore \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias my-key-alias

# Sign APK
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore my-release-key.keystore app/build/outputs/apk/release/app-release-unsigned.apk \
  my-key-alias

# Optimize with zipalign
zipalign -v 4 app-release-unsigned.apk app-release-signed.apk
```

---

## 🌐 Publish to Play Store

1. **Prepare Assets**
   - Icon: 512×512 PNG
   - Screenshots: 5-8 images
   - Description: 80 char title, 4000 char description

2. **Create App on Console**
   - https://play.google.com/console
   - Create new app → "Real-Time Caption"

3. **Upload APK**
   - Release → Production
   - Upload signed APK
   - Fill metadata, rating, privacy policy

4. **Beta Testing**
   - Release → Internal Testing
   - Add test accounts, monitor crashes

5. **Launch**
   - 1-2% staged rollout
   - Monitor crash rate, reviews
   - Scale to 100% over 1-2 weeks

---

## 📞 Support & Debugging

### Common Errors

| Error | Solution |
|-------|----------|
| `Cannot find whisper-tiny-en.tflite` | Download model to `app/src/main/assets/` |
| `ML Kit translation timeout` | First run: wait for model download (Wi-Fi) |
| `Overlay not showing` | Grant SYSTEM_ALERT_WINDOW in Settings |
| `MediaProjection permission denied` | Tap "Allow" in system dialog |
| `Audio not captured` | Play system audio (YouTube, Spotify) |
| `High battery drain` | Reduce AUDIO_BUFFER_WINDOW or disable GPU |

### Get Help
- **GitHub Issues**: https://github.com/yourorg/RealTimeCaptionApp/issues
- **Stack Overflow**: Tag `android`, `tensorflow-lite`, `ml-kit`
- **Email**: support@yourcompany.com

---

## 🎓 Learning Resources

### Whisper
- [OpenAI Whisper GitHub](https://github.com/openai/whisper)
- [Whisper Paper](https://arxiv.org/pdf/2212.04356.pdf)

### TensorFlow Lite
- [Official Docs](https://tensorflow.org/lite)
- [Mobile Performance](https://tensorflow.org/lite/performance)

### Android Audio
- [AudioPlaybackCapture](https://developer.android.com/reference/android/media/AudioPlaybackCaptureConfiguration)
- [AudioRecord API](https://developer.android.com/reference/android/media/AudioRecord)

### Kotlin Coroutines
- [Coroutines Guide](https://kotlinlang.org/docs/coroutines-overview.html)
- [Flow Guide](https://kotlinlang.org/docs/flow.html)

---

## 🎯 Next Steps

1. **First Build**: Follow steps 1-5 above
2. **Test Manually**: Play audio, watch captions
3. **Customize**: Change colors, fonts, languages
4. **Deploy**: Sign APK and upload to Play Store
5. **Monitor**: Track crashes, user feedback

---

**Questions?** Check [README.md](README.md) or [ARCHITECTURE.md](ARCHITECTURE.md)

**Ready to ship?** See [BUILD & DEPLOY](BUILD_DEPLOY.md) guide
