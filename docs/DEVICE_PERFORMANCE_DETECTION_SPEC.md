# Device Performance Detection Specification

## Overview

This document specifies the Device Performance Detection system for Plunderstorm Mobile, which automatically detects device capabilities and configures optimal graphics settings for the best possible gameplay experience on each device.

## System Architecture

### Detection Flow

```
App Launch
    │
    ▼
┌─────────────────────┐
│ Check Cached Profile│ ─── Found ───► Use Cached Settings
└─────────────────────┘
    │ Not Found
    ▼
┌─────────────────────┐
│  Collect Device Info│
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  Check Known Device │ ─── Found ───► Use Known Profile
│      Database       │
└─────────────────────┘
    │ Not Found
    ▼
┌─────────────────────┐
│  Run Benchmark      │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│ Calculate Settings  │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐
│  Cache & Apply      │
└─────────────────────┘
```

## Device Information Collection

### Hardware Detection

```typescript
interface DeviceInfo {
  // Platform
  platform: 'ios' | 'android';
  osVersion: string;
  deviceModel: string;
  manufacturer: string;

  // Display
  screenWidth: number;
  screenHeight: number;
  screenDensity: number;
  refreshRate: number;
  hdrSupported: boolean;

  // CPU
  cpuCores: number;
  cpuArchitecture: string;
  cpuFrequency?: number;        // MHz if available

  // GPU
  gpuVendor: string;
  gpuRenderer: string;
  gpuVersion: string;
  maxTextureSize: number;
  supportedExtensions: string[];

  // Memory
  totalRAM: number;             // MB
  availableRAM: number;         // MB
  totalStorage: number;         // MB
  availableStorage: number;     // MB

  // Battery
  batteryLevel: number;         // 0-100
  isCharging: boolean;
  batteryHealth?: string;

  // Thermal
  thermalState: ThermalState;
}

enum ThermalState {
  NOMINAL = 'nominal',
  FAIR = 'fair',
  SERIOUS = 'serious',
  CRITICAL = 'critical'
}
```

### Platform-Specific Collection

```typescript
// iOS
async function collectIOSDeviceInfo(): Promise<DeviceInfo> {
  const device = await getIOSDeviceInfo();

  return {
    platform: 'ios',
    osVersion: device.systemVersion,
    deviceModel: device.model,          // "iPhone14,2"
    manufacturer: 'Apple',
    screenWidth: device.screenWidth,
    screenHeight: device.screenHeight,
    screenDensity: device.screenScale,
    refreshRate: device.maximumFramesPerSecond,
    hdrSupported: device.hdrSupported,
    cpuCores: device.processorCount,
    cpuArchitecture: device.cpuArchitecture,
    gpuVendor: 'Apple',
    gpuRenderer: device.gpuFamily,      // "Apple GPU Family 8"
    gpuVersion: device.metalVersion,
    maxTextureSize: device.maxTextureSize,
    supportedExtensions: device.gpuFeatures,
    totalRAM: device.physicalMemory,
    availableRAM: device.availableMemory,
    totalStorage: device.totalDiskSpace,
    availableStorage: device.freeDiskSpace,
    batteryLevel: device.batteryLevel * 100,
    isCharging: device.batteryState === 'charging',
    thermalState: mapThermalState(device.thermalState)
  };
}

// Android
async function collectAndroidDeviceInfo(): Promise<DeviceInfo> {
  const device = await getAndroidDeviceInfo();

  return {
    platform: 'android',
    osVersion: device.sdkVersion.toString(),
    deviceModel: device.model,
    manufacturer: device.manufacturer,
    screenWidth: device.displayMetrics.widthPixels,
    screenHeight: device.displayMetrics.heightPixels,
    screenDensity: device.displayMetrics.density,
    refreshRate: device.display.refreshRate,
    hdrSupported: device.display.hdrCapabilities !== null,
    cpuCores: Runtime.getRuntime().availableProcessors(),
    cpuArchitecture: device.supportedAbis[0],
    gpuVendor: device.glVendor,
    gpuRenderer: device.glRenderer,
    gpuVersion: device.glVersion,
    maxTextureSize: device.glMaxTextureSize,
    supportedExtensions: device.glExtensions,
    totalRAM: device.totalMemory,
    availableRAM: device.availableMemory,
    totalStorage: device.totalStorage,
    availableStorage: device.availableStorage,
    batteryLevel: device.batteryLevel,
    isCharging: device.isCharging,
    thermalState: mapAndroidThermalState(device.thermalStatus)
  };
}
```

## Known Device Database

### Device Profiles

```typescript
interface KnownDeviceProfile {
  devicePattern: string;        // Regex pattern for model
  tier: PerformanceTier;
  recommendedSettings: GraphicsSettings;
  specialFlags?: DeviceFlags;
}

enum PerformanceTier {
  ULTRA = 'ultra',
  HIGH = 'high',
  MEDIUM = 'medium',
  LOW = 'low',
  MINIMUM = 'minimum'
}

interface DeviceFlags {
  thermalIssues?: boolean;
  batteryDrain?: boolean;
  gpuBugs?: string[];
  forceSettings?: Partial<GraphicsSettings>;
}

const KNOWN_DEVICE_PROFILES: KnownDeviceProfile[] = [
  // iOS Devices
  {
    devicePattern: 'iPhone15,.*',   // iPhone 14 Pro / Pro Max
    tier: PerformanceTier.ULTRA,
    recommendedSettings: ULTRA_SETTINGS
  },
  {
    devicePattern: 'iPhone14,.*',   // iPhone 13 series
    tier: PerformanceTier.HIGH,
    recommendedSettings: HIGH_SETTINGS
  },
  {
    devicePattern: 'iPhone12,.*',   // iPhone 11 series
    tier: PerformanceTier.MEDIUM,
    recommendedSettings: MEDIUM_SETTINGS
  },
  {
    devicePattern: 'iPhone10,.*',   // iPhone X/8 series
    tier: PerformanceTier.LOW,
    recommendedSettings: LOW_SETTINGS
  },
  {
    devicePattern: 'iPad13,.*',     // iPad Pro M1/M2
    tier: PerformanceTier.ULTRA,
    recommendedSettings: ULTRA_SETTINGS
  },

  // Android Devices - Flagship
  {
    devicePattern: 'SM-S9.*',       // Samsung Galaxy S23 series
    tier: PerformanceTier.ULTRA,
    recommendedSettings: ULTRA_SETTINGS
  },
  {
    devicePattern: 'SM-S90.*',      // Samsung Galaxy S22 series
    tier: PerformanceTier.HIGH,
    recommendedSettings: HIGH_SETTINGS
  },
  {
    devicePattern: 'Pixel 8.*',
    tier: PerformanceTier.HIGH,
    recommendedSettings: HIGH_SETTINGS
  },
  {
    devicePattern: 'Pixel 7.*',
    tier: PerformanceTier.MEDIUM,
    recommendedSettings: MEDIUM_SETTINGS
  },

  // Android Devices - Mid-range
  {
    devicePattern: 'SM-A5.*',       // Samsung Galaxy A5x series
    tier: PerformanceTier.MEDIUM,
    recommendedSettings: MEDIUM_SETTINGS
  },
  {
    devicePattern: 'Redmi Note 1.*',
    tier: PerformanceTier.MEDIUM,
    recommendedSettings: MEDIUM_SETTINGS
  },

  // Android Devices - Budget
  {
    devicePattern: 'SM-A1.*',       // Samsung Galaxy A1x series
    tier: PerformanceTier.LOW,
    recommendedSettings: LOW_SETTINGS
  }
];

function findKnownProfile(deviceModel: string): KnownDeviceProfile | null {
  for (const profile of KNOWN_DEVICE_PROFILES) {
    const regex = new RegExp(profile.devicePattern);
    if (regex.test(deviceModel)) {
      return profile;
    }
  }
  return null;
}
```

## Benchmark System

### Benchmark Tests

```typescript
interface BenchmarkSuite {
  tests: BenchmarkTest[];
  totalDuration: number;        // ms target
  requiredTests: string[];      // Must pass these
}

interface BenchmarkTest {
  id: string;
  name: string;
  type: 'cpu' | 'gpu' | 'memory' | 'io';
  weight: number;               // Importance for scoring
  timeout: number;              // ms
  run: () => Promise<BenchmarkResult>;
}

interface BenchmarkResult {
  testId: string;
  score: number;                // 0-100
  duration: number;             // ms
  metrics: Record<string, number>;
}

const BENCHMARK_SUITE: BenchmarkSuite = {
  tests: [
    {
      id: 'cpu_math',
      name: 'CPU Math Operations',
      type: 'cpu',
      weight: 20,
      timeout: 2000,
      run: runCPUMathBenchmark
    },
    {
      id: 'gpu_triangles',
      name: 'GPU Triangle Rendering',
      type: 'gpu',
      weight: 30,
      timeout: 3000,
      run: runGPUTriangleBenchmark
    },
    {
      id: 'gpu_textures',
      name: 'GPU Texture Sampling',
      type: 'gpu',
      weight: 25,
      timeout: 3000,
      run: runGPUTextureBenchmark
    },
    {
      id: 'memory_bandwidth',
      name: 'Memory Bandwidth',
      type: 'memory',
      weight: 15,
      timeout: 2000,
      run: runMemoryBenchmark
    },
    {
      id: 'io_speed',
      name: 'Storage I/O Speed',
      type: 'io',
      weight: 10,
      timeout: 2000,
      run: runIOBenchmark
    }
  ],
  totalDuration: 10000,
  requiredTests: ['gpu_triangles', 'gpu_textures']
};
```

### Benchmark Implementation

```typescript
async function runCPUMathBenchmark(): Promise<BenchmarkResult> {
  const startTime = performance.now();
  const iterations = 1000000;

  // Matrix operations
  let result = 0;
  for (let i = 0; i < iterations; i++) {
    result += Math.sin(i) * Math.cos(i) * Math.tan(i % 1.5);
  }

  const duration = performance.now() - startTime;
  const expectedTime = 500;     // Expected for mid-range device
  const score = Math.min(100, Math.round((expectedTime / duration) * 50));

  return {
    testId: 'cpu_math',
    score,
    duration,
    metrics: { iterations, opsPerSecond: iterations / (duration / 1000) }
  };
}

async function runGPUTriangleBenchmark(): Promise<BenchmarkResult> {
  const renderer = createBenchmarkRenderer();
  const triangleCount = 50000;
  const frames = 60;

  const frameTimes: number[] = [];
  for (let f = 0; f < frames; f++) {
    const frameStart = performance.now();

    renderer.clear();
    for (let t = 0; t < triangleCount; t++) {
      renderer.drawTriangle(getRandomTriangle());
    }
    renderer.present();

    frameTimes.push(performance.now() - frameStart);
  }

  const avgFrameTime = frameTimes.reduce((a, b) => a + b) / frames;
  const fps = 1000 / avgFrameTime;

  // Score based on achieved FPS (60 = 100, 30 = 50, etc.)
  const score = Math.min(100, Math.round((fps / 60) * 100));

  renderer.destroy();

  return {
    testId: 'gpu_triangles',
    score,
    duration: frameTimes.reduce((a, b) => a + b),
    metrics: { triangleCount, avgFrameTime, fps }
  };
}

async function runGPUTextureBenchmark(): Promise<BenchmarkResult> {
  const renderer = createBenchmarkRenderer();
  const textureSize = 1024;
  const sampleCount = 100000;
  const frames = 30;

  // Create test texture
  const texture = renderer.createTexture(textureSize, textureSize);

  const frameTimes: number[] = [];
  for (let f = 0; f < frames; f++) {
    const frameStart = performance.now();

    for (let s = 0; s < sampleCount; s++) {
      renderer.sampleTexture(texture, Math.random(), Math.random());
    }
    renderer.present();

    frameTimes.push(performance.now() - frameStart);
  }

  const avgFrameTime = frameTimes.reduce((a, b) => a + b) / frames;
  const samplesPerSecond = sampleCount / (avgFrameTime / 1000);

  const score = Math.min(100, Math.round((samplesPerSecond / 10000000) * 100));

  texture.destroy();
  renderer.destroy();

  return {
    testId: 'gpu_textures',
    score,
    duration: frameTimes.reduce((a, b) => a + b),
    metrics: { textureSize, sampleCount, samplesPerSecond }
  };
}
```

### Benchmark Score Calculation

```typescript
interface BenchmarkScore {
  overall: number;              // 0-100
  tier: PerformanceTier;
  breakdown: {
    cpu: number;
    gpu: number;
    memory: number;
    io: number;
  };
}

function calculateBenchmarkScore(results: BenchmarkResult[]): BenchmarkScore {
  const breakdown = {
    cpu: 0,
    gpu: 0,
    memory: 0,
    io: 0
  };

  let totalWeight = 0;
  let weightedSum = 0;

  for (const result of results) {
    const test = BENCHMARK_SUITE.tests.find(t => t.id === result.testId);
    if (!test) continue;

    // Update breakdown
    breakdown[test.type] = Math.max(breakdown[test.type], result.score);

    // Calculate weighted score
    weightedSum += result.score * test.weight;
    totalWeight += test.weight;
  }

  const overall = Math.round(weightedSum / totalWeight);
  const tier = scoreTiTier(overall);

  return { overall, tier, breakdown };
}

function scoreTiTier(score: number): PerformanceTier {
  if (score >= 90) return PerformanceTier.ULTRA;
  if (score >= 70) return PerformanceTier.HIGH;
  if (score >= 50) return PerformanceTier.MEDIUM;
  if (score >= 30) return PerformanceTier.LOW;
  return PerformanceTier.MINIMUM;
}
```

## Graphics Settings

### Settings Presets

```typescript
interface GraphicsSettings {
  // Quality
  preset: PerformanceTier;

  // Resolution
  renderScale: number;          // 0.5 - 1.0
  targetResolution: 'native' | '1080p' | '720p' | '540p';

  // Frame Rate
  targetFPS: 30 | 60 | 90 | 120;
  vsync: boolean;
  frameRateCap: boolean;

  // Textures
  textureQuality: 'ultra' | 'high' | 'medium' | 'low';
  textureFiltering: 'trilinear' | 'bilinear' | 'point';
  mipmaps: boolean;

  // Effects
  shadowQuality: 'ultra' | 'high' | 'medium' | 'low' | 'off';
  shadowResolution: 2048 | 1024 | 512 | 256;
  particleQuality: 'high' | 'medium' | 'low';
  particleDensity: number;      // 0.25 - 1.0

  // Post-processing
  antiAliasing: 'TAA' | 'FXAA' | 'off';
  bloom: boolean;
  ambientOcclusion: boolean;
  motionBlur: boolean;
  depthOfField: boolean;

  // Lighting
  dynamicLighting: boolean;
  reflections: 'SSR' | 'cubemap' | 'off';
  waterQuality: 'high' | 'medium' | 'low';

  // Draw Distance
  viewDistance: 'far' | 'medium' | 'near';
  lodBias: number;              // 0 - 2
  grassDensity: number;         // 0 - 1

  // Audio (affected by performance)
  audioChannels: number;
  spatialAudio: boolean;
}

const ULTRA_SETTINGS: GraphicsSettings = {
  preset: PerformanceTier.ULTRA,
  renderScale: 1.0,
  targetResolution: 'native',
  targetFPS: 60,
  vsync: true,
  frameRateCap: false,
  textureQuality: 'ultra',
  textureFiltering: 'trilinear',
  mipmaps: true,
  shadowQuality: 'ultra',
  shadowResolution: 2048,
  particleQuality: 'high',
  particleDensity: 1.0,
  antiAliasing: 'TAA',
  bloom: true,
  ambientOcclusion: true,
  motionBlur: true,
  depthOfField: true,
  dynamicLighting: true,
  reflections: 'SSR',
  waterQuality: 'high',
  viewDistance: 'far',
  lodBias: 0,
  grassDensity: 1.0,
  audioChannels: 32,
  spatialAudio: true
};

const HIGH_SETTINGS: GraphicsSettings = {
  preset: PerformanceTier.HIGH,
  renderScale: 1.0,
  targetResolution: '1080p',
  targetFPS: 60,
  vsync: true,
  frameRateCap: true,
  textureQuality: 'high',
  textureFiltering: 'trilinear',
  mipmaps: true,
  shadowQuality: 'high',
  shadowResolution: 1024,
  particleQuality: 'high',
  particleDensity: 0.8,
  antiAliasing: 'FXAA',
  bloom: true,
  ambientOcclusion: true,
  motionBlur: false,
  depthOfField: false,
  dynamicLighting: true,
  reflections: 'cubemap',
  waterQuality: 'high',
  viewDistance: 'far',
  lodBias: 0.5,
  grassDensity: 0.8,
  audioChannels: 24,
  spatialAudio: true
};

const MEDIUM_SETTINGS: GraphicsSettings = {
  preset: PerformanceTier.MEDIUM,
  renderScale: 0.85,
  targetResolution: '720p',
  targetFPS: 60,
  vsync: true,
  frameRateCap: true,
  textureQuality: 'medium',
  textureFiltering: 'bilinear',
  mipmaps: true,
  shadowQuality: 'medium',
  shadowResolution: 512,
  particleQuality: 'medium',
  particleDensity: 0.6,
  antiAliasing: 'FXAA',
  bloom: false,
  ambientOcclusion: false,
  motionBlur: false,
  depthOfField: false,
  dynamicLighting: true,
  reflections: 'cubemap',
  waterQuality: 'medium',
  viewDistance: 'medium',
  lodBias: 1.0,
  grassDensity: 0.5,
  audioChannels: 16,
  spatialAudio: false
};

const LOW_SETTINGS: GraphicsSettings = {
  preset: PerformanceTier.LOW,
  renderScale: 0.75,
  targetResolution: '540p',
  targetFPS: 30,
  vsync: true,
  frameRateCap: true,
  textureQuality: 'low',
  textureFiltering: 'bilinear',
  mipmaps: false,
  shadowQuality: 'low',
  shadowResolution: 256,
  particleQuality: 'low',
  particleDensity: 0.4,
  antiAliasing: 'off',
  bloom: false,
  ambientOcclusion: false,
  motionBlur: false,
  depthOfField: false,
  dynamicLighting: false,
  reflections: 'off',
  waterQuality: 'low',
  viewDistance: 'near',
  lodBias: 1.5,
  grassDensity: 0.2,
  audioChannels: 8,
  spatialAudio: false
};

const MINIMUM_SETTINGS: GraphicsSettings = {
  preset: PerformanceTier.MINIMUM,
  renderScale: 0.5,
  targetResolution: '540p',
  targetFPS: 30,
  vsync: true,
  frameRateCap: true,
  textureQuality: 'low',
  textureFiltering: 'point',
  mipmaps: false,
  shadowQuality: 'off',
  shadowResolution: 256,
  particleQuality: 'low',
  particleDensity: 0.25,
  antiAliasing: 'off',
  bloom: false,
  ambientOcclusion: false,
  motionBlur: false,
  depthOfField: false,
  dynamicLighting: false,
  reflections: 'off',
  waterQuality: 'low',
  viewDistance: 'near',
  lodBias: 2.0,
  grassDensity: 0,
  audioChannels: 4,
  spatialAudio: false
};
```

## Thermal Management

### Thermal Monitoring

```typescript
interface ThermalManager {
  currentState: ThermalState;
  temperature?: number;
  throttlingActive: boolean;
  listeners: ThermalListener[];
}

interface ThermalListener {
  onStateChange: (state: ThermalState) => void;
  onThrottling: (throttled: boolean) => void;
}

class ThermalMonitor {
  private manager: ThermalManager;
  private checkInterval: number = 5000;  // 5 seconds

  start(): void {
    setInterval(() => this.checkThermalState(), this.checkInterval);
  }

  private async checkThermalState(): void {
    const state = await getPlatformThermalState();

    if (state !== this.manager.currentState) {
      this.manager.currentState = state;
      this.notifyStateChange(state);

      // Auto-adjust settings if needed
      if (state === ThermalState.SERIOUS || state === ThermalState.CRITICAL) {
        this.activateThrottling();
      } else if (state === ThermalState.NOMINAL) {
        this.deactivateThrottling();
      }
    }
  }

  private activateThrottling(): void {
    if (this.manager.throttlingActive) return;

    this.manager.throttlingActive = true;

    // Reduce settings
    const currentSettings = getCurrentSettings();
    const reducedSettings = reduceSettingsForThermal(currentSettings);
    applySettings(reducedSettings);

    // Notify user
    showNotification('Performance reduced to prevent overheating');

    this.notifyThrottling(true);
  }

  private deactivateThrottling(): void {
    if (!this.manager.throttlingActive) return;

    this.manager.throttlingActive = false;

    // Restore settings
    const userSettings = getUserPreferredSettings();
    applySettings(userSettings);

    this.notifyThrottling(false);
  }
}

function reduceSettingsForThermal(current: GraphicsSettings): GraphicsSettings {
  return {
    ...current,
    targetFPS: Math.min(current.targetFPS, 30),
    renderScale: Math.min(current.renderScale, 0.75),
    particleDensity: current.particleDensity * 0.5,
    shadowQuality: current.shadowQuality === 'ultra' ? 'high' :
                   current.shadowQuality === 'high' ? 'medium' :
                   current.shadowQuality === 'medium' ? 'low' : 'off'
  };
}
```

## Battery Optimization

### Battery Monitoring

```typescript
interface BatteryManager {
  level: number;
  isCharging: boolean;
  lowPowerMode: boolean;
  batterySaverActive: boolean;
}

interface BatteryConfig {
  lowBatteryThreshold: 20;      // %
  criticalBatteryThreshold: 10; // %
  autoSaverEnabled: boolean;
  autoSaverThreshold: number;   // % to activate
}

class BatteryMonitor {
  private manager: BatteryManager;
  private config: BatteryConfig = {
    lowBatteryThreshold: 20,
    criticalBatteryThreshold: 10,
    autoSaverEnabled: true,
    autoSaverThreshold: 20
  };

  async checkBattery(): Promise<void> {
    const status = await getBatteryStatus();

    this.manager.level = status.level;
    this.manager.isCharging = status.charging;
    this.manager.lowPowerMode = status.lowPowerMode;

    // Auto-enable battery saver
    if (this.config.autoSaverEnabled && !this.manager.isCharging) {
      if (status.level <= this.config.autoSaverThreshold && !this.manager.batterySaverActive) {
        this.enableBatterySaver();
      } else if (status.level > this.config.autoSaverThreshold + 5 && this.manager.batterySaverActive) {
        // Hysteresis to prevent toggling
        this.disableBatterySaver();
      }
    }

    // Show warnings
    if (status.level <= this.config.criticalBatteryThreshold && !status.charging) {
      showCriticalBatteryWarning();
    } else if (status.level <= this.config.lowBatteryThreshold && !status.charging) {
      showLowBatteryWarning();
    }
  }

  enableBatterySaver(): void {
    this.manager.batterySaverActive = true;

    const batterySaverSettings: Partial<GraphicsSettings> = {
      targetFPS: 30,
      renderScale: 0.75,
      particleDensity: 0.5,
      shadowQuality: 'low',
      bloom: false,
      ambientOcclusion: false,
      dynamicLighting: false
    };

    applySettingsOverride(batterySaverSettings);
    showNotification('Battery Saver activated');
  }

  disableBatterySaver(): void {
    this.manager.batterySaverActive = false;
    removeSettingsOverride();
    showNotification('Battery Saver deactivated');
  }
}
```

## Settings UI

### Auto-Detection Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│  GRAPHICS SETTINGS                                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  DETECTED PERFORMANCE: HIGH                                             │
│  Device: iPhone 14 Pro                                                  │
│  GPU Score: 85/100                                                      │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  RECOMMENDED SETTINGS                                           │   │
│  │                                                                 │   │
│  │  • Resolution: 1080p native                                     │   │
│  │  • Frame Rate: 60 FPS                                          │   │
│  │  • Shadows: High                                               │   │
│  │  • Effects: High                                               │   │
│  │                                                                 │   │
│  │  [Apply Recommended]                                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  CURRENT PRESET: [Ultra ▼]                                             │
│                                                                         │
│  ⚠️ Your device may experience heating at Ultra settings.              │
│                                                                         │
│  [Customize Settings]        [Re-run Benchmark]                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Benchmark UI

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    PERFORMANCE BENCHMARK                                │
│                                                                         │
│  Running tests to detect optimal settings...                           │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │  [████████████████░░░░░░░░░░]  GPU Test 2/3                    │   │
│  │                                                                 │   │
│  │  Testing texture sampling performance...                       │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  Results so far:                                                        │
│  ✓ CPU: 78/100                                                         │
│  ✓ GPU Triangles: 85/100                                               │
│  ⏳ GPU Textures: Running...                                           │
│  ○ Memory: Pending                                                     │
│  ○ Storage: Pending                                                    │
│                                                                         │
│                    [Cancel]                                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Minimum Requirements

### Minimum Specs Warning

```typescript
interface MinimumRequirements {
  ramMin: 2048;                 // 2 GB
  storageMin: 2048;             // 2 GB
  gpuScoreMin: 20;
  osVersionMin: {
    ios: '14.0',
    android: 24                 // Android 7.0
  };
}

function checkMinimumRequirements(device: DeviceInfo): RequirementsCheck {
  const issues: string[] = [];

  if (device.totalRAM < MINIMUM_REQUIREMENTS.ramMin) {
    issues.push(`Insufficient RAM: ${device.totalRAM}MB (need ${MINIMUM_REQUIREMENTS.ramMin}MB)`);
  }

  if (device.availableStorage < MINIMUM_REQUIREMENTS.storageMin) {
    issues.push(`Insufficient storage: ${device.availableStorage}MB (need ${MINIMUM_REQUIREMENTS.storageMin}MB)`);
  }

  // Check OS version
  if (device.platform === 'ios') {
    if (compareVersions(device.osVersion, MINIMUM_REQUIREMENTS.osVersionMin.ios) < 0) {
      issues.push(`iOS version too old: ${device.osVersion} (need ${MINIMUM_REQUIREMENTS.osVersionMin.ios})`);
    }
  } else {
    if (parseInt(device.osVersion) < MINIMUM_REQUIREMENTS.osVersionMin.android) {
      issues.push(`Android version too old (need Android 7.0+)`);
    }
  }

  return {
    meetRequirements: issues.length === 0,
    issues,
    canPlay: issues.length === 0 || !issues.some(i => i.includes('version'))
  };
}
```

### Requirements Warning UI

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    ⚠️ DEVICE WARNING                                   │
│                                                                         │
│  Your device may not provide the best experience.                      │
│                                                                         │
│  Issues detected:                                                       │
│  • Low available RAM (1.5 GB)                                          │
│  • Limited storage space (1.2 GB free)                                 │
│                                                                         │
│  You can still play, but may experience:                               │
│  • Longer loading times                                                │
│  • Occasional stuttering                                               │
│  • Reduced visual quality                                              │
│                                                                         │
│            [Continue Anyway]        [Exit]                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Analytics Events

```typescript
// Device detection
analytics.track('device_detected', {
  platform: string,
  model: string,
  osVersion: string,
  ramMB: number,
  gpuRenderer: string,
  tier: PerformanceTier,
  benchmarkScore: number,
  usedKnownProfile: boolean
});

// Settings applied
analytics.track('graphics_settings_applied', {
  preset: string,
  targetFPS: number,
  renderScale: number,
  shadowQuality: string,
  customized: boolean
});

// Performance issues
analytics.track('performance_issue', {
  type: 'thermal' | 'battery' | 'fps_drop',
  severity: string,
  actionTaken: string
});
```

## Testing Requirements

### Functional Tests

- [ ] Device info collected correctly on iOS
- [ ] Device info collected correctly on Android
- [ ] Known device profiles match correctly
- [ ] Benchmark runs and produces scores
- [ ] Settings applied correctly per tier
- [ ] Thermal throttling activates correctly
- [ ] Battery saver activates correctly
- [ ] Settings UI displays recommendations

### Device Coverage Tests

- [ ] Test on flagship iOS devices
- [ ] Test on flagship Android devices
- [ ] Test on mid-range devices
- [ ] Test on budget devices
- [ ] Test on tablets

### Edge Cases

- [ ] Unknown device falls back to benchmark
- [ ] Benchmark timeout handled
- [ ] Very low scores show minimum settings
- [ ] Storage warning at 90%+ usage

## Success Metrics

| Metric | Target |
|--------|--------|
| Auto-detection accuracy | 90%+ correct tier |
| First-time settings acceptance | 70%+ keep recommended |
| Performance complaints | < 5% of users |
| Thermal throttling events | < 10% of sessions |

## Dependencies

- **UX-015**: Settings Menu
- **UX-023**: Battery Optimization
- **BACK-003**: Analytics System
