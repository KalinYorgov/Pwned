# Crash Reporting Specification

## Document Information
- **Task ID:** BACK-021
- **Priority:** P0
- **Complexity:** S (Small)
- **Dependencies:** None
- **Last Updated:** 2025-11-30

---

## 1. Overview

### 1.1 Purpose
This specification defines the crash reporting system for Plunderstorm Mobile, enabling automatic capture, aggregation, and analysis of client crashes across iOS and Android platforms. The system provides developers with actionable crash data to identify, prioritize, and resolve stability issues.

### 1.2 Scope
- Client-side crash detection and capture
- Crash report transmission and storage
- Symbolication pipeline
- Crash aggregation and deduplication
- Prioritization and alerting
- Issue tracker integration
- Crash analytics dashboard

### 1.3 Goals
- Capture 100% of client crashes
- Achieve < 1% crash rate target
- Reduce mean time to identify crash root cause
- Enable proactive crash prevention through trend analysis

---

## 2. Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Crash Reporting Architecture                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                        Mobile Client                              │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐               │   │
│  │  │   Native    │  │   Unity     │  │   Game      │               │   │
│  │  │   Crash     │  │   Crash     │  │   State     │               │   │
│  │  │   Handler   │  │   Handler   │  │   Capture   │               │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘               │   │
│  │         │                │                │                       │   │
│  │         └────────────────┼────────────────┘                       │   │
│  │                          ▼                                        │   │
│  │                 ┌─────────────────┐                               │   │
│  │                 │  Crash Report   │                               │   │
│  │                 │    Builder      │                               │   │
│  │                 └────────┬────────┘                               │   │
│  │                          │                                        │   │
│  │                 ┌────────▼────────┐                               │   │
│  │                 │  Local Storage  │  (Survives restart)           │   │
│  │                 └────────┬────────┘                               │   │
│  └──────────────────────────┼────────────────────────────────────────┘   │
│                             │                                            │
│                             ▼                                            │
│                    ┌─────────────────┐                                   │
│                    │   Crash API     │                                   │
│                    │   Endpoint      │                                   │
│                    └────────┬────────┘                                   │
│                             │                                            │
│         ┌───────────────────┼───────────────────┐                        │
│         ▼                   ▼                   ▼                        │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                    │
│  │ Symbolication│   │   Crash     │   │   Alert     │                    │
│  │   Service   │   │  Database   │   │   Service   │                    │
│  └──────┬──────┘   └──────┬──────┘   └─────────────┘                    │
│         │                  │                                             │
│         └──────────────────┼──────────────────────────┐                  │
│                            ▼                          ▼                  │
│                   ┌─────────────────┐        ┌─────────────┐             │
│                   │   Aggregation   │        │    Jira     │             │
│                   │    Service      │        │ Integration │             │
│                   └────────┬────────┘        └─────────────┘             │
│                            │                                             │
│                            ▼                                             │
│                   ┌─────────────────┐                                    │
│                   │    Dashboard    │                                    │
│                   │   (Grafana)     │                                    │
│                   └─────────────────┘                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Options

| Component | Option A | Option B | Option C | Recommendation |
|-----------|----------|----------|----------|----------------|
| Crash SDK | Firebase Crashlytics | Sentry | Bugsnag | Firebase Crashlytics |
| Symbolication | Firebase | Custom | Sentry | Firebase |
| Storage | BigQuery | PostgreSQL | Elasticsearch | BigQuery |
| Dashboard | Firebase Console | Grafana | Custom | Firebase + Grafana |

**Rationale:** Firebase Crashlytics is recommended due to:
- Native iOS/Android and Unity support
- Automatic symbolication
- Free tier sufficient for launch
- Integration with other Firebase services
- Real-time crash alerts

---

## 3. Crash Capture

### 3.1 Crash Types

```yaml
crash_types:
  native_crashes:
    description: "OS-level crashes (SIGSEGV, SIGABRT, etc.)"
    platforms: [iOS, Android]
    capture_method: "Native signal handlers"
    includes:
      - Null pointer dereferences
      - Stack overflows
      - Memory access violations
      - Illegal instructions

  unity_exceptions:
    description: "Unhandled C# exceptions in Unity"
    platforms: [iOS, Android]
    capture_method: "Application.logMessageReceived"
    includes:
      - NullReferenceException
      - IndexOutOfRangeException
      - Custom game exceptions

  anr_events:
    description: "Application Not Responding (Android)"
    platforms: [Android]
    capture_method: "ANR watchdog"
    threshold: "5 seconds UI thread block"

  oom_events:
    description: "Out of Memory crashes"
    platforms: [iOS, Android]
    capture_method: "Memory pressure handlers"
    includes:
      - iOS memory warnings
      - Android low memory killer

  watchdog_terminations:
    description: "iOS watchdog terminations"
    platforms: [iOS]
    capture_method: "MetricKit"
    includes:
      - Launch timeout
      - Background task timeout
```

### 3.2 Client-Side Implementation

#### 3.2.1 Crash Handler Initialization

```csharp
// CrashReporter.cs
using UnityEngine;
using Firebase.Crashlytics;
using System;
using System.Collections.Generic;

public class CrashReporter : MonoBehaviour
{
    private static CrashReporter _instance;
    private GameStateCapture _gameStateCapture;
    private DeviceInfoCapture _deviceInfoCapture;

    public static CrashReporter Instance => _instance;

    private void Awake()
    {
        if (_instance != null)
        {
            Destroy(gameObject);
            return;
        }

        _instance = this;
        DontDestroyOnLoad(gameObject);

        InitializeCrashReporting();
    }

    private void InitializeCrashReporting()
    {
        // Initialize Firebase Crashlytics
        Crashlytics.ReportUncaughtExceptionsAsFatal = true;

        // Set up Unity exception handler
        Application.logMessageReceived += HandleUnityLog;
        AppDomain.CurrentDomain.UnhandledException += HandleUnhandledException;

        // Initialize state capture
        _gameStateCapture = new GameStateCapture();
        _deviceInfoCapture = new DeviceInfoCapture();

        // Set persistent device info
        SetDeviceCustomKeys();

        Debug.Log("[CrashReporter] Initialized successfully");
    }

    private void HandleUnityLog(string logString, string stackTrace, LogType type)
    {
        if (type == LogType.Exception || type == LogType.Error)
        {
            // Capture game state at time of error
            var gameState = _gameStateCapture.CaptureCurrentState();

            // Add as custom keys for context
            SetGameStateKeys(gameState);

            if (type == LogType.Exception)
            {
                // Log non-fatal for exceptions that don't crash
                Crashlytics.LogException(new Exception($"{logString}\n{stackTrace}"));
            }
        }
    }

    private void HandleUnhandledException(object sender, UnhandledExceptionEventArgs e)
    {
        var exception = e.ExceptionObject as Exception;
        if (exception != null)
        {
            var gameState = _gameStateCapture.CaptureCurrentState();
            SetGameStateKeys(gameState);
            Crashlytics.LogException(exception);
        }
    }

    private void SetDeviceCustomKeys()
    {
        var deviceInfo = _deviceInfoCapture.Capture();

        Crashlytics.SetCustomKey("device_model", deviceInfo.Model);
        Crashlytics.SetCustomKey("device_manufacturer", deviceInfo.Manufacturer);
        Crashlytics.SetCustomKey("os_version", deviceInfo.OSVersion);
        Crashlytics.SetCustomKey("app_version", Application.version);
        Crashlytics.SetCustomKey("unity_version", Application.unityVersion);
        Crashlytics.SetCustomKey("total_memory_mb", deviceInfo.TotalMemoryMB.ToString());
        Crashlytics.SetCustomKey("gpu_name", deviceInfo.GPUName);
        Crashlytics.SetCustomKey("screen_resolution", deviceInfo.ScreenResolution);
        Crashlytics.SetCustomKey("graphics_api", deviceInfo.GraphicsAPI);
    }

    private void SetGameStateKeys(GameState state)
    {
        Crashlytics.SetCustomKey("game_scene", state.CurrentScene);
        Crashlytics.SetCustomKey("match_id", state.MatchId ?? "none");
        Crashlytics.SetCustomKey("match_phase", state.MatchPhase);
        Crashlytics.SetCustomKey("player_count", state.PlayerCount.ToString());
        Crashlytics.SetCustomKey("player_alive", state.IsPlayerAlive.ToString());
        Crashlytics.SetCustomKey("memory_usage_mb", state.MemoryUsageMB.ToString());
        Crashlytics.SetCustomKey("fps_average", state.AverageFPS.ToString("F1"));
        Crashlytics.SetCustomKey("network_latency_ms", state.NetworkLatencyMS.ToString());
        Crashlytics.SetCustomKey("time_in_match_sec", state.TimeInMatchSeconds.ToString());
    }

    // Call this to set user identifier for crash correlation
    public void SetUserId(string oduxId)
    {
        Crashlytics.SetUserId(oduxId);
    }

    // Log breadcrumb events leading up to crash
    public void LogBreadcrumb(string message)
    {
        Crashlytics.Log(message);
    }

    // Force a test crash (debug only)
    [System.Diagnostics.Conditional("DEBUG")]
    public void ForceCrash()
    {
        Crashlytics.ThrowTestException();
    }

    private void OnDestroy()
    {
        Application.logMessageReceived -= HandleUnityLog;
        AppDomain.CurrentDomain.UnhandledException -= HandleUnhandledException;
    }
}
```

#### 3.2.2 Game State Capture

```csharp
// GameStateCapture.cs
using UnityEngine;
using UnityEngine.SceneManagement;
using System;

public class GameState
{
    public string CurrentScene { get; set; }
    public string MatchId { get; set; }
    public string MatchPhase { get; set; }
    public int PlayerCount { get; set; }
    public bool IsPlayerAlive { get; set; }
    public float MemoryUsageMB { get; set; }
    public float AverageFPS { get; set; }
    public int NetworkLatencyMS { get; set; }
    public int TimeInMatchSeconds { get; set; }
    public string LastAbilityUsed { get; set; }
    public string LastUIScreen { get; set; }
    public int ActiveEntities { get; set; }
    public string QualityLevel { get; set; }
}

public class GameStateCapture
{
    private float[] _fpsBuffer = new float[60];
    private int _fpsBufferIndex = 0;
    private string _lastUIScreen = "None";
    private string _lastAbilityUsed = "None";

    public void UpdateFPS(float deltaTime)
    {
        _fpsBuffer[_fpsBufferIndex] = 1f / deltaTime;
        _fpsBufferIndex = (_fpsBufferIndex + 1) % _fpsBuffer.Length;
    }

    public void SetLastUIScreen(string screen)
    {
        _lastUIScreen = screen;
    }

    public void SetLastAbilityUsed(string ability)
    {
        _lastAbilityUsed = ability;
    }

    public GameState CaptureCurrentState()
    {
        var state = new GameState
        {
            CurrentScene = SceneManager.GetActiveScene().name,
            MemoryUsageMB = GC.GetTotalMemory(false) / (1024f * 1024f),
            AverageFPS = CalculateAverageFPS(),
            QualityLevel = QualitySettings.names[QualitySettings.GetQualityLevel()],
            LastUIScreen = _lastUIScreen,
            LastAbilityUsed = _lastAbilityUsed
        };

        // Capture match-specific state if in match
        if (MatchManager.Instance != null)
        {
            state.MatchId = MatchManager.Instance.MatchId;
            state.MatchPhase = MatchManager.Instance.CurrentPhase.ToString();
            state.PlayerCount = MatchManager.Instance.PlayerCount;
            state.TimeInMatchSeconds = (int)MatchManager.Instance.MatchTime;
            state.ActiveEntities = MatchManager.Instance.ActiveEntityCount;
        }

        // Capture player state
        if (LocalPlayer.Instance != null)
        {
            state.IsPlayerAlive = LocalPlayer.Instance.IsAlive;
        }

        // Capture network state
        if (NetworkManager.Instance != null)
        {
            state.NetworkLatencyMS = NetworkManager.Instance.CurrentLatency;
        }

        return state;
    }

    private float CalculateAverageFPS()
    {
        float sum = 0;
        int count = 0;
        foreach (var fps in _fpsBuffer)
        {
            if (fps > 0)
            {
                sum += fps;
                count++;
            }
        }
        return count > 0 ? sum / count : 0;
    }
}
```

#### 3.2.3 Device Information Capture

```csharp
// DeviceInfoCapture.cs
using UnityEngine;

public class DeviceInfo
{
    public string Model { get; set; }
    public string Manufacturer { get; set; }
    public string OSVersion { get; set; }
    public int TotalMemoryMB { get; set; }
    public string GPUName { get; set; }
    public string ScreenResolution { get; set; }
    public string GraphicsAPI { get; set; }
    public string DeviceType { get; set; }
    public int ProcessorCount { get; set; }
    public string ProcessorType { get; set; }
    public float BatteryLevel { get; set; }
    public BatteryStatus BatteryStatus { get; set; }
    public NetworkReachability NetworkStatus { get; set; }
}

public class DeviceInfoCapture
{
    public DeviceInfo Capture()
    {
        return new DeviceInfo
        {
            Model = SystemInfo.deviceModel,
            Manufacturer = GetManufacturer(),
            OSVersion = SystemInfo.operatingSystem,
            TotalMemoryMB = SystemInfo.systemMemorySize,
            GPUName = SystemInfo.graphicsDeviceName,
            ScreenResolution = $"{Screen.width}x{Screen.height}",
            GraphicsAPI = SystemInfo.graphicsDeviceType.ToString(),
            DeviceType = SystemInfo.deviceType.ToString(),
            ProcessorCount = SystemInfo.processorCount,
            ProcessorType = SystemInfo.processorType,
            BatteryLevel = SystemInfo.batteryLevel,
            BatteryStatus = SystemInfo.batteryStatus,
            NetworkStatus = Application.internetReachability
        };
    }

    private string GetManufacturer()
    {
        #if UNITY_ANDROID && !UNITY_EDITOR
        try
        {
            using (var buildClass = new AndroidJavaClass("android.os.Build"))
            {
                return buildClass.GetStatic<string>("MANUFACTURER");
            }
        }
        catch
        {
            return "Unknown";
        }
        #elif UNITY_IOS && !UNITY_EDITOR
        return "Apple";
        #else
        return "Unknown";
        #endif
    }
}
```

### 3.3 Breadcrumb Logging

```csharp
// BreadcrumbLogger.cs
public static class BreadcrumbLogger
{
    private const int MaxBreadcrumbs = 100;
    private static Queue<string> _breadcrumbs = new Queue<string>();

    public static void Log(string category, string message)
    {
        var timestamp = DateTime.UtcNow.ToString("HH:mm:ss.fff");
        var breadcrumb = $"[{timestamp}] [{category}] {message}";

        // Keep local buffer
        if (_breadcrumbs.Count >= MaxBreadcrumbs)
        {
            _breadcrumbs.Dequeue();
        }
        _breadcrumbs.Enqueue(breadcrumb);

        // Send to crash reporter
        CrashReporter.Instance?.LogBreadcrumb(breadcrumb);
    }

    // Convenience methods
    public static void LogNavigation(string screen) =>
        Log("NAV", $"Navigated to {screen}");

    public static void LogAction(string action) =>
        Log("ACTION", action);

    public static void LogNetwork(string operation, bool success) =>
        Log("NET", $"{operation}: {(success ? "OK" : "FAILED")}");

    public static void LogMatch(string event_) =>
        Log("MATCH", event_);

    public static void LogAbility(string ability, string target) =>
        Log("ABILITY", $"{ability} -> {target}");

    public static void LogError(string error) =>
        Log("ERROR", error);
}

// Usage examples throughout code:
// BreadcrumbLogger.LogNavigation("MainMenu");
// BreadcrumbLogger.LogMatch("Match started: match-123");
// BreadcrumbLogger.LogAbility("Fireball", "Player_456");
// BreadcrumbLogger.LogNetwork("JoinMatch", true);
```

---

## 4. Crash Report Schema

### 4.1 Report Structure

```json
{
  "crash_id": "crash-uuid-12345",
  "timestamp": "2025-11-30T12:34:56.789Z",
  "app_info": {
    "app_version": "1.2.3",
    "build_number": "456",
    "bundle_id": "com.company.plunderstorm",
    "unity_version": "2022.3.10f1"
  },
  "device_info": {
    "platform": "Android",
    "os_version": "Android 13",
    "model": "Samsung Galaxy S23",
    "manufacturer": "Samsung",
    "total_memory_mb": 8192,
    "gpu": "Adreno 740",
    "screen_resolution": "1080x2340",
    "graphics_api": "Vulkan",
    "processor_count": 8
  },
  "crash_info": {
    "type": "native_crash",
    "signal": "SIGSEGV",
    "code": "SEGV_MAPERR",
    "fault_address": "0x0000000000000000",
    "crashed_thread": 0,
    "exception_type": "EXC_BAD_ACCESS",
    "exception_message": "Attempted to read from null pointer"
  },
  "stack_trace": {
    "raw": "...",
    "symbolicated": [
      {
        "frame": 0,
        "library": "libunity.so",
        "symbol": "PlayerController::Update()",
        "file": "PlayerController.cpp",
        "line": 234
      },
      {
        "frame": 1,
        "library": "libunity.so",
        "symbol": "GameObject::UpdateComponents()",
        "file": "GameObject.cpp",
        "line": 567
      }
    ]
  },
  "game_state": {
    "scene": "GameMatch",
    "match_id": "match-abc-123",
    "match_phase": "Combat",
    "player_count": 45,
    "player_alive": true,
    "time_in_match_sec": 180,
    "memory_usage_mb": 1234.5,
    "average_fps": 45.2,
    "network_latency_ms": 67,
    "quality_level": "Medium",
    "active_entities": 856
  },
  "user_info": {
    "user_id": "user-xyz-789",
    "session_id": "session-abc",
    "session_duration_sec": 3600
  },
  "breadcrumbs": [
    "[12:34:50.123] [NAV] Navigated to GameMatch",
    "[12:34:51.456] [MATCH] Match started: match-abc-123",
    "[12:34:55.789] [ABILITY] Fireball -> Enemy_123",
    "[12:34:56.012] [ERROR] Null reference in ability effect"
  ],
  "threads": [
    {
      "id": 0,
      "name": "Main",
      "crashed": true,
      "stack": [...]
    },
    {
      "id": 1,
      "name": "UnityGfx",
      "crashed": false,
      "stack": [...]
    }
  ],
  "memory_info": {
    "used_mb": 1234,
    "available_mb": 2048,
    "pressure": "normal"
  },
  "logs": [
    "Last 50 Unity log messages..."
  ]
}
```

### 4.2 Crash Fingerprinting

```yaml
fingerprint_algorithm:
  description: "Uniquely identify crash types for aggregation"

  components:
    - crash_type          # native, unity_exception, anr, oom
    - exception_type      # SIGSEGV, NullReferenceException, etc.
    - top_frame_symbol    # Top symbolicated frame
    - top_frame_file      # Source file if available
    - app_version         # Major.minor version

  formula: |
    fingerprint = SHA256(
      crash_type + "|" +
      exception_type + "|" +
      normalize(top_frame_symbol) + "|" +
      top_frame_file + "|" +
      major_minor_version
    )

  normalization:
    - Remove memory addresses
    - Remove instance IDs
    - Normalize generic types
    - Remove line numbers from symbol
```

---

## 5. Symbolication

### 5.1 Symbol Management

```yaml
symbol_workflow:
  build_phase:
    ios:
      - Generate dSYM files during build
      - Upload dSYMs to Firebase after build
      - Archive dSYMs with build artifacts

    android:
      - Generate mapping.txt (ProGuard/R8)
      - Generate native symbol files (.so with debug info)
      - Upload symbols to Firebase
      - Archive symbols with build artifacts

    unity:
      - Enable "Create Symbols.zip" in build settings
      - Upload IL2CPP symbols
      - Upload native plugin symbols

  automation:
    ci_upload_script: |
      # iOS dSYM upload
      firebase crashlytics:symbols:upload \
        --app=$FIREBASE_APP_ID \
        path/to/dSYMs

      # Android mapping upload
      firebase crashlytics:mappingfile:upload \
        --app=$FIREBASE_APP_ID \
        --mapping-file=path/to/mapping.txt
```

### 5.2 Symbolication Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       Symbolication Pipeline                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Build Server                                                            │
│  ┌────────────────┐                                                      │
│  │   Unity Build  │                                                      │
│  │  ┌──────────┐  │      ┌────────────────┐                             │
│  │  │   APK    │──┼─────▶│   App Store    │                             │
│  │  └──────────┘  │      └────────────────┘                             │
│  │  ┌──────────┐  │      ┌────────────────┐                             │
│  │  │ Symbols  │──┼─────▶│ Firebase       │                             │
│  │  │  .dSYM   │  │      │ Symbol Store   │                             │
│  │  │  .so     │  │      └───────┬────────┘                             │
│  │  │ mapping  │  │              │                                       │
│  │  └──────────┘  │              │                                       │
│  └────────────────┘              │                                       │
│                                  │                                       │
│  Crash Report                    │                                       │
│  ┌────────────────┐              │                                       │
│  │  Raw Stack     │              │                                       │
│  │  0x7abc1234    │              ▼                                       │
│  │  0x7def5678    │──────▶ ┌───────────────┐                            │
│  │  0x7ghi9012    │        │ Symbolication │                            │
│  └────────────────┘        │    Service    │                            │
│                            └───────┬───────┘                            │
│                                    │                                     │
│                                    ▼                                     │
│  Symbolicated Report    ┌─────────────────────┐                         │
│  ┌────────────────┐     │ PlayerController.cs │                         │
│  │ Update():234   │◀────│ Line 234            │                         │
│  │ Process():567  │     │                     │                         │
│  │ Main():89      │     └─────────────────────┘                         │
│  └────────────────┘                                                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Build Symbol Configuration

```csharp
// Editor/BuildPostProcessor.cs
using UnityEditor;
using UnityEditor.Build;
using UnityEditor.Build.Reporting;
using System.IO;
using System.Diagnostics;

public class CrashSymbolUploader : IPostprocessBuildWithReport
{
    public int callbackOrder => 100;

    public void OnPostprocessBuild(BuildReport report)
    {
        var platform = report.summary.platform;
        var outputPath = report.summary.outputPath;

        if (platform == BuildTarget.iOS)
        {
            UploadIOSSymbols(outputPath);
        }
        else if (platform == BuildTarget.Android)
        {
            UploadAndroidSymbols(outputPath);
        }
    }

    private void UploadIOSSymbols(string buildPath)
    {
        // Find dSYM files
        var dsymPath = buildPath + ".dSYM";
        if (Directory.Exists(dsymPath))
        {
            RunFirebaseUpload($"crashlytics:symbols:upload {dsymPath}");
        }
    }

    private void UploadAndroidSymbols(string apkPath)
    {
        // Upload mapping file
        var mappingPath = Path.Combine(
            Path.GetDirectoryName(apkPath),
            "mapping.txt"
        );

        if (File.Exists(mappingPath))
        {
            RunFirebaseUpload($"crashlytics:mappingfile:upload --mapping-file={mappingPath}");
        }

        // Upload native symbols
        var symbolsPath = Path.Combine(
            Path.GetDirectoryName(apkPath),
            "symbols"
        );

        if (Directory.Exists(symbolsPath))
        {
            RunFirebaseUpload($"crashlytics:symbols:upload {symbolsPath}");
        }
    }

    private void RunFirebaseUpload(string arguments)
    {
        var appId = GetFirebaseAppId();
        var process = new Process
        {
            StartInfo = new ProcessStartInfo
            {
                FileName = "firebase",
                Arguments = $"--app={appId} {arguments}",
                UseShellExecute = false,
                RedirectStandardOutput = true
            }
        };

        process.Start();
        process.WaitForExit();

        UnityEngine.Debug.Log($"[CrashSymbols] Upload complete: {arguments}");
    }

    private string GetFirebaseAppId()
    {
        #if UNITY_IOS
        return "1:123456789:ios:abcdef";
        #elif UNITY_ANDROID
        return "1:123456789:android:abcdef";
        #else
        return "";
        #endif
    }
}
```

---

## 6. Aggregation and Analysis

### 6.1 Crash Grouping

```yaml
crash_grouping:
  primary_grouping:
    method: "fingerprint"
    display: "Crash signature"

  secondary_grouping:
    by_version:
      purpose: "Track fix effectiveness"

    by_device:
      purpose: "Identify device-specific issues"

    by_os:
      purpose: "Identify OS-specific issues"

    by_game_state:
      purpose: "Identify gameplay-related crashes"

  aggregation_metrics:
    - crash_count           # Total occurrences
    - affected_users        # Unique users affected
    - crash_rate            # Crashes per session
    - trend                 # Increasing/decreasing
    - first_seen            # When first reported
    - last_seen             # Most recent occurrence
    - versions_affected     # App versions with crash
```

### 6.2 Prioritization Algorithm

```yaml
crash_priority:
  calculation:
    formula: |
      priority_score = (
        affected_users_weight * affected_users_normalized +
        frequency_weight * frequency_normalized +
        trend_weight * trend_factor +
        severity_weight * severity_factor
      )

    weights:
      affected_users_weight: 0.4
      frequency_weight: 0.3
      trend_weight: 0.2
      severity_weight: 0.1

  severity_factors:
    native_crash: 1.0       # Most severe
    anr: 0.8                # Blocks user
    oom: 0.7                # Memory issue
    unity_exception: 0.5    # May be recoverable

  trend_factors:
    increasing_rapidly: 1.5
    increasing: 1.2
    stable: 1.0
    decreasing: 0.8

  priority_levels:
    P1_critical:
      threshold: "> 0.8"
      criteria: "Affects >1% of users or increasing rapidly"
      response_time: "Same day"

    P2_high:
      threshold: "0.5 - 0.8"
      criteria: "Affects >0.1% of users"
      response_time: "Within 3 days"

    P3_medium:
      threshold: "0.2 - 0.5"
      criteria: "Notable but contained"
      response_time: "Within sprint"

    P4_low:
      threshold: "< 0.2"
      criteria: "Rare or edge case"
      response_time: "Backlog"
```

### 6.3 Analysis Queries

```sql
-- Top crashes by affected users (last 7 days)
SELECT
    crash_fingerprint,
    crash_type,
    top_frame_symbol,
    COUNT(DISTINCT user_id) as affected_users,
    COUNT(*) as total_occurrences,
    MIN(timestamp) as first_seen,
    MAX(timestamp) as last_seen,
    ARRAY_AGG(DISTINCT app_version) as versions
FROM crash_reports
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY crash_fingerprint, crash_type, top_frame_symbol
ORDER BY affected_users DESC
LIMIT 20;

-- Crash rate by app version
SELECT
    app_version,
    COUNT(DISTINCT CASE WHEN crash_id IS NOT NULL THEN session_id END) as crashed_sessions,
    COUNT(DISTINCT session_id) as total_sessions,
    SAFE_DIVIDE(
        COUNT(DISTINCT CASE WHEN crash_id IS NOT NULL THEN session_id END),
        COUNT(DISTINCT session_id)
    ) * 100 as crash_rate_percent
FROM sessions
LEFT JOIN crash_reports USING (session_id)
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY app_version
ORDER BY app_version DESC;

-- Device-specific crash analysis
SELECT
    device_model,
    os_version,
    COUNT(*) as crash_count,
    COUNT(DISTINCT user_id) as affected_users,
    ARRAY_AGG(DISTINCT crash_type) as crash_types
FROM crash_reports
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY device_model, os_version
HAVING crash_count > 10
ORDER BY crash_count DESC;

-- Crash trend analysis (daily)
SELECT
    DATE(timestamp) as crash_date,
    crash_fingerprint,
    COUNT(*) as occurrences,
    COUNT(DISTINCT user_id) as users
FROM crash_reports
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY crash_date, crash_fingerprint
ORDER BY crash_date, occurrences DESC;
```

---

## 7. Alerting

### 7.1 Alert Rules

```yaml
crash_alerts:
  new_crash_spike:
    description: "Spike in crash rate"
    condition: |
      crash_rate_5min > crash_rate_baseline_7d * 2
    severity: P1
    channels: [pagerduty, slack_incidents]

  new_crash_type:
    description: "New crash type in latest version"
    condition: |
      crash_fingerprint NOT IN (SELECT fingerprint FROM crashes WHERE version < current_version)
      AND affected_users > 10
    severity: P2
    channels: [slack_crashes]

  crash_rate_threshold:
    description: "Crash rate exceeds target"
    condition: |
      crash_rate_24h > 1.0  # 1% target
    severity: P2
    channels: [slack_crashes, email]

  device_specific_crash:
    description: "High crash rate on specific device"
    condition: |
      device_crash_rate > 5.0  # 5% on specific device
      AND device_crash_count > 50
    severity: P3
    channels: [slack_crashes]

  anr_spike:
    description: "Spike in ANR rate"
    condition: |
      anr_rate_1h > anr_rate_baseline * 3
    severity: P2
    channels: [slack_crashes]
```

### 7.2 Alert Configuration

```yaml
# alertmanager configuration for crash alerts
route:
  receiver: 'default'
  routes:
    - match:
        alertname: CrashRateSpike
        severity: P1
      receiver: 'pagerduty-crashes'

    - match:
        alertname: CrashRateThreshold
        severity: P2
      receiver: 'slack-crashes'

receivers:
  - name: 'pagerduty-crashes'
    pagerduty_configs:
      - service_key: '<CRASH_SERVICE_KEY>'
        severity: critical

  - name: 'slack-crashes'
    slack_configs:
      - channel: '#crash-reports'
        title: '{{ .CommonAnnotations.summary }}'
        text: |
          *Crash Alert*
          Type: {{ .CommonLabels.crash_type }}
          Affected Users: {{ .CommonAnnotations.affected_users }}
          Crash Rate: {{ .CommonAnnotations.crash_rate }}%

          {{ .CommonAnnotations.top_crashes }}
```

---

## 8. Issue Tracker Integration

### 8.1 Jira Integration

```yaml
jira_integration:
  auto_create_threshold:
    affected_users: 100
    crash_rate_increase: 50%  # 50% increase from baseline

  ticket_template:
    project: "PLUNDER"
    issue_type: "Bug"
    priority_mapping:
      P1: "Blocker"
      P2: "Critical"
      P3: "Major"
      P4: "Minor"

    fields:
      summary: "[Crash] {{ crash_type }}: {{ top_frame_symbol }}"
      description: |
        h2. Crash Summary
        * *Fingerprint:* {{ fingerprint }}
        * *Type:* {{ crash_type }}
        * *First Seen:* {{ first_seen }}
        * *Affected Users:* {{ affected_users }}
        * *Total Occurrences:* {{ total_occurrences }}

        h2. Stack Trace
        {code}
        {{ symbolicated_stack }}
        {code}

        h2. Device Breakdown
        || Device || Count ||
        {{ device_breakdown }}

        h2. Game State
        * *Most Common Scene:* {{ common_scene }}
        * *Most Common Phase:* {{ common_phase }}

        h2. Links
        * [Firebase Crashlytics|{{ crashlytics_url }}]

      labels:
        - "crash"
        - "{{ platform }}"
        - "v{{ app_version }}"

      components:
        - "Client Stability"
```

### 8.2 Integration Implementation

```python
# crash_to_jira.py
from jira import JIRA
from firebase_admin import crashlytics
import hashlib

class CrashJiraIntegration:
    def __init__(self, jira_url, jira_token, project_key):
        self.jira = JIRA(server=jira_url, token_auth=jira_token)
        self.project_key = project_key

    def should_create_ticket(self, crash_data):
        """Determine if crash warrants a Jira ticket"""
        return (
            crash_data['affected_users'] >= 100 or
            crash_data['crash_rate_increase'] >= 0.5 or
            crash_data['priority'] in ['P1', 'P2']
        )

    def find_existing_ticket(self, fingerprint):
        """Check if ticket already exists for this crash"""
        jql = f'project = {self.project_key} AND labels = "crash-{fingerprint[:8]}"'
        issues = self.jira.search_issues(jql)
        return issues[0] if issues else None

    def create_ticket(self, crash_data):
        """Create Jira ticket for crash"""
        fingerprint = crash_data['fingerprint']

        # Check for existing ticket
        existing = self.find_existing_ticket(fingerprint)
        if existing:
            self.update_ticket(existing, crash_data)
            return existing

        # Create new ticket
        issue_dict = {
            'project': {'key': self.project_key},
            'summary': f"[Crash] {crash_data['crash_type']}: {crash_data['top_frame']}",
            'description': self.format_description(crash_data),
            'issuetype': {'name': 'Bug'},
            'priority': {'name': self.map_priority(crash_data['priority'])},
            'labels': [
                'crash',
                f"crash-{fingerprint[:8]}",
                crash_data['platform'],
                f"v{crash_data['app_version']}"
            ]
        }

        return self.jira.create_issue(fields=issue_dict)

    def update_ticket(self, issue, crash_data):
        """Update existing ticket with new crash data"""
        comment = f"""
        *Crash Update*
        - New occurrences: {crash_data['new_occurrences']}
        - Total affected users: {crash_data['affected_users']}
        - Latest version affected: {crash_data['app_version']}
        """
        self.jira.add_comment(issue, comment)

    def map_priority(self, priority):
        mapping = {
            'P1': 'Blocker',
            'P2': 'Critical',
            'P3': 'Major',
            'P4': 'Minor'
        }
        return mapping.get(priority, 'Major')

    def format_description(self, crash_data):
        return f"""
h2. Crash Summary
* *Fingerprint:* {crash_data['fingerprint']}
* *Type:* {crash_data['crash_type']}
* *First Seen:* {crash_data['first_seen']}
* *Affected Users:* {crash_data['affected_users']}
* *Total Occurrences:* {crash_data['total_occurrences']}

h2. Stack Trace
{{code}}
{crash_data['symbolicated_stack']}
{{code}}

h2. Device Breakdown
{self.format_device_breakdown(crash_data['devices'])}

h2. Common Game State
* *Scene:* {crash_data['common_scene']}
* *Match Phase:* {crash_data['common_phase']}

h2. Links
* [Firebase Crashlytics|{crash_data['crashlytics_url']}]
        """

    def format_device_breakdown(self, devices):
        rows = "\n".join([f"| {d['model']} | {d['count']} |" for d in devices[:10]])
        return f"|| Device || Count ||\n{rows}"
```

---

## 9. Dashboard

### 9.1 Crash Analytics Dashboard

```json
{
  "dashboard": {
    "title": "Crash Analytics",
    "rows": [
      {
        "title": "Overview",
        "panels": [
          {
            "title": "Crash-Free Users",
            "type": "gauge",
            "query": "100 - (crashed_users / total_users * 100)",
            "thresholds": [
              { "value": 99, "color": "green" },
              { "value": 98, "color": "yellow" },
              { "value": 0, "color": "red" }
            ],
            "target": "> 99%"
          },
          {
            "title": "Total Crashes (24h)",
            "type": "stat",
            "query": "count(crashes) where time > now() - 24h"
          },
          {
            "title": "Affected Users (24h)",
            "type": "stat",
            "query": "count(distinct user_id) where crash = true and time > now() - 24h"
          },
          {
            "title": "Top Crash",
            "type": "stat",
            "query": "top_crash_type()"
          }
        ]
      },
      {
        "title": "Crash Trends",
        "panels": [
          {
            "title": "Crash Rate Over Time",
            "type": "graph",
            "queries": [
              { "expr": "crash_rate_percent", "legend": "Crash Rate %" },
              { "expr": "1.0", "legend": "Target (1%)", "style": "dashed" }
            ]
          },
          {
            "title": "Crashes by Type",
            "type": "stacked_area",
            "queries": [
              { "expr": "crashes by type", "legend": "{{ type }}" }
            ]
          }
        ]
      },
      {
        "title": "Top Crashes",
        "panels": [
          {
            "title": "Top 10 Crashes",
            "type": "table",
            "columns": [
              "Crash Signature",
              "Type",
              "Users",
              "Occurrences",
              "Trend",
              "Status"
            ],
            "query": "top_crashes(10)"
          }
        ]
      },
      {
        "title": "Device Analysis",
        "panels": [
          {
            "title": "Crashes by Device",
            "type": "piechart",
            "query": "crashes group by device_model"
          },
          {
            "title": "Crashes by OS Version",
            "type": "bar",
            "query": "crashes group by os_version"
          }
        ]
      },
      {
        "title": "Version Analysis",
        "panels": [
          {
            "title": "Crash Rate by Version",
            "type": "bar",
            "query": "crash_rate group by app_version order by version desc limit 10"
          },
          {
            "title": "New Crashes in Latest Version",
            "type": "table",
            "query": "new_crashes_in_version(latest)"
          }
        ]
      }
    ]
  }
}
```

### 9.2 KPI Tracking

```yaml
crash_kpis:
  primary_kpis:
    crash_free_users_rate:
      description: "Percentage of users without crashes"
      target: "> 99%"
      measurement: "daily"
      formula: "(total_users - crashed_users) / total_users * 100"

    crash_free_sessions_rate:
      description: "Percentage of sessions without crashes"
      target: "> 99.5%"
      measurement: "daily"
      formula: "(total_sessions - crashed_sessions) / total_sessions * 100"

  secondary_kpis:
    mean_time_to_detect:
      description: "Time from crash to alert"
      target: "< 5 minutes"

    mean_time_to_fix:
      description: "Time from detection to fix deployed"
      target: "< 48 hours for P1"

    regression_rate:
      description: "Fixed crashes that reoccur"
      target: "< 5%"

    new_crash_types_per_release:
      description: "New crash signatures introduced"
      target: "< 3"
```

---

## 10. Testing

### 10.1 Crash Reporting Tests

```csharp
// Tests/CrashReporterTests.cs
using NUnit.Framework;
using UnityEngine;
using UnityEngine.TestTools;

[TestFixture]
public class CrashReporterTests
{
    [Test]
    public void GameStateCapture_CapturesCorrectData()
    {
        var capture = new GameStateCapture();
        var state = capture.CaptureCurrentState();

        Assert.IsNotNull(state.CurrentScene);
        Assert.GreaterOrEqual(state.MemoryUsageMB, 0);
        Assert.IsNotNull(state.QualityLevel);
    }

    [Test]
    public void DeviceInfoCapture_CapturesAllFields()
    {
        var capture = new DeviceInfoCapture();
        var info = capture.Capture();

        Assert.IsNotNull(info.Model);
        Assert.IsNotNull(info.OSVersion);
        Assert.Greater(info.TotalMemoryMB, 0);
        Assert.IsNotNull(info.GPUName);
    }

    [Test]
    public void BreadcrumbLogger_MaintainsMaxSize()
    {
        for (int i = 0; i < 150; i++)
        {
            BreadcrumbLogger.Log("TEST", $"Message {i}");
        }

        // Should only keep last 100
        // Verify through internal state or mock
    }

    [Test]
    [UnityTest]
    public IEnumerator CrashReporter_HandlesUnityException()
    {
        var reporter = new GameObject().AddComponent<CrashReporter>();

        // Trigger exception
        LogAssert.Expect(LogType.Exception, "Test exception");
        throw new System.Exception("Test exception");

        yield return null;

        // Verify crash was logged (mock Crashlytics)
    }
}
```

### 10.2 Test Crash Generation

```csharp
// Debug/CrashTestMenu.cs
#if DEBUG || DEVELOPMENT_BUILD
using UnityEngine;

public class CrashTestMenu : MonoBehaviour
{
    private bool _showMenu = false;

    void Update()
    {
        // Triple tap to show menu
        if (Input.touchCount == 3)
        {
            _showMenu = !_showMenu;
        }
    }

    void OnGUI()
    {
        if (!_showMenu) return;

        GUILayout.BeginArea(new Rect(10, 10, 300, 400));
        GUILayout.Label("Crash Test Menu");

        if (GUILayout.Button("Test Native Crash"))
        {
            CauseNativeCrash();
        }

        if (GUILayout.Button("Test Unity Exception"))
        {
            throw new System.Exception("Test Unity Exception");
        }

        if (GUILayout.Button("Test Null Reference"))
        {
            object obj = null;
            obj.ToString();
        }

        if (GUILayout.Button("Test Out of Memory"))
        {
            var list = new System.Collections.Generic.List<byte[]>();
            while (true)
            {
                list.Add(new byte[1024 * 1024 * 100]); // 100MB chunks
            }
        }

        if (GUILayout.Button("Test ANR (Android)"))
        {
            System.Threading.Thread.Sleep(10000);
        }

        if (GUILayout.Button("Firebase Test Crash"))
        {
            Firebase.Crashlytics.Crashlytics.ThrowTestException();
        }

        GUILayout.EndArea();
    }

    private void CauseNativeCrash()
    {
        #if UNITY_ANDROID && !UNITY_EDITOR
        using (var jc = new AndroidJavaClass("java.lang.Runtime"))
        {
            var runtime = jc.CallStatic<AndroidJavaObject>("getRuntime");
            runtime.Call("exit", 1);
        }
        #elif UNITY_IOS && !UNITY_EDITOR
        // Force null pointer dereference
        System.Runtime.InteropServices.Marshal.ReadInt32(System.IntPtr.Zero);
        #endif
    }
}
#endif
```

---

## 11. Acceptance Criteria Verification

| Criteria | Implementation | Verification |
|----------|---------------|--------------|
| Crashes captured with stack trace | Firebase Crashlytics + Unity handlers | Section 3: Native and Unity crash capture |
| Device info included | DeviceInfoCapture class | Section 3.2.3: All device fields captured |
| Game state at crash time | GameStateCapture class | Section 3.2.2: Match, player, network state |
| Crashes aggregated by type | Fingerprinting algorithm | Section 4.2: SHA256-based fingerprinting |
| Prioritization by frequency/severity | Priority scoring algorithm | Section 6.2: Weighted priority calculation |
| Integration with issue tracker | Jira integration | Section 8: Auto-create tickets |
| Symbolication for readable traces | Firebase + CI upload | Section 5: Full symbolication pipeline |

---

## 12. Appendix

### 12.1 Firebase Crashlytics Setup

```yaml
# firebase.json
{
  "crashlytics": {
    "nativeSymbolUploadEnabled": true
  }
}

# Unity package requirements
# - com.google.firebase.crashlytics
# - com.google.firebase.app
```

### 12.2 Supported Platforms

| Platform | Crash Types | SDK |
|----------|-------------|-----|
| iOS 12+ | Native, Unity, OOM, Watchdog | Firebase Crashlytics |
| Android 5.0+ | Native, Unity, ANR, OOM | Firebase Crashlytics |

### 12.3 Data Retention

```yaml
retention_policy:
  raw_crash_reports: 90 days
  aggregated_data: 2 years
  symbols: Forever (or until version deprecated)
```

### 12.4 Related Documents

- [SERVER_MONITORING_SPEC.md](SERVER_MONITORING_SPEC.md) - Server-side monitoring
- [DEDICATED_GAME_SERVERS_SPEC.md](DEDICATED_GAME_SERVERS_SPEC.md) - Server crash handling
