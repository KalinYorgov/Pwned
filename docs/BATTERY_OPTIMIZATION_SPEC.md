# Battery Optimization Specification

## Document Information
- **Task ID:** UX-023
- **Priority:** P2
- **Complexity:** Small
- **Status:** Complete
- **Last Updated:** 2025-12-06

---

## Overview

This document specifies battery optimization features for Plunderstorm Mobile, enabling extended play sessions on mobile devices while maintaining acceptable gameplay quality. Mobile gaming is power-intensive, and providing battery-saving options improves user experience and device longevity.

---

## Battery Saver Mode

### Mode Configuration

```typescript
const BATTERY_SAVER_MODE = {
  description: 'Reduced power consumption mode for extended play',

  activation: {
    manual: {
      location: 'Settings > Performance > Battery Saver',
      quickAccess: 'Settings gear in main menu',
      toggle: true
    },
    automatic: {
      enabled: true,
      threshold: 20, // percent
      promptUser: true,
      promptMessage: 'Battery low. Enable Battery Saver mode?',
      promptOptions: ['Enable', 'Not Now', 'Never Ask']
    }
  },

  settings: {
    enabled: {
      type: 'toggle',
      default: false
    },
    autoActivate: {
      type: 'toggle',
      default: true,
      label: 'Auto-enable at low battery'
    },
    autoThreshold: {
      type: 'slider',
      range: [10, 30],
      default: 20,
      unit: '%',
      label: 'Auto-enable threshold'
    }
  }
};
```

### Graphics Reductions

```typescript
const BATTERY_SAVER_GRAPHICS = {
  frameRate: {
    normal: 60,
    batterySaver: 30,
    reduction: '50% fewer frames rendered',
    smoothTransition: true,
    transitionDuration: 500 // ms
  },

  resolution: {
    normal: 'Native or 90%',
    batterySaver: '70% of native',
    method: 'Render scale reduction',
    upscaling: 'Bilinear filtering'
  },

  shadows: {
    normal: 'Medium-High quality',
    batterySaver: 'Disabled or blob shadows only',
    savings: 'Significant GPU reduction'
  },

  postProcessing: {
    normal: {
      bloom: true,
      antiAliasing: 'FXAA or MSAA 2x',
      ambientOcclusion: true,
      motionBlur: 'Optional'
    },
    batterySaver: {
      bloom: false,
      antiAliasing: 'Disabled',
      ambientOcclusion: false,
      motionBlur: false
    }
  },

  particles: {
    normal: '100% particle count',
    batterySaver: '50% particle count',
    affectedSystems: [
      'Environmental particles (dust, leaves)',
      'Ability VFX (reduced density)',
      'Weather effects'
    ],
    excludedSystems: [
      'Critical gameplay indicators',
      'Damage numbers',
      'UI particles'
    ]
  },

  textures: {
    normal: 'High resolution',
    batterySaver: 'Medium resolution',
    mipMapBias: '+1 level in battery saver'
  },

  drawDistance: {
    normal: '100%',
    batterySaver: '80%',
    affectsLOD: true,
    lodTransitionDistance: 'Reduced by 20%'
  },

  lighting: {
    normal: 'Dynamic lighting',
    batterySaver: 'Simplified lighting model',
    realtimeLights: 'Reduced from 4 to 2 max'
  }
};
```

### Performance Impact Estimates

```typescript
const BATTERY_SAVINGS_ESTIMATES = {
  disclaimer: 'Actual savings vary by device and usage',

  estimated: {
    frameRateReduction: {
      savings: '30-40%',
      reason: 'Half the frames = significant GPU/CPU reduction'
    },
    resolutionReduction: {
      savings: '15-25%',
      reason: 'Fewer pixels to shade'
    },
    shadowsDisabled: {
      savings: '10-15%',
      reason: 'Shadow maps are expensive'
    },
    postProcessDisabled: {
      savings: '5-10%',
      reason: 'Full-screen effects eliminated'
    },
    particlesReduced: {
      savings: '5-8%',
      reason: 'Fewer particles to simulate and render'
    }
  },

  totalEstimated: {
    combinedSavings: '40-60% power reduction',
    playTimeIncrease: '1.5-2x longer sessions',
    note: 'Most significant saving is from FPS reduction'
  }
};
```

---

## Battery Level Display

### UI Integration

```typescript
const BATTERY_LEVEL_UI = {
  display: {
    location: 'Top status bar area',
    showByDefault: false,
    setting: {
      name: 'Show Battery Level',
      type: 'toggle',
      default: false,
      location: 'Settings > Performance'
    }
  },

  indicator: {
    type: 'Icon with percentage',
    icon: {
      full: 'battery_full',
      medium: 'battery_medium', // 50-80%
      low: 'battery_low', // 20-50%
      critical: 'battery_critical', // <20%
      charging: 'battery_charging'
    },
    percentage: {
      show: true,
      format: '{value}%',
      fontSize: 12
    },
    position: {
      anchor: 'top-right',
      offset: { x: -10, y: 10 },
      afterWifiIndicator: true
    }
  },

  colors: {
    full: '#4CAF50', // Green
    medium: '#8BC34A', // Light green
    low: '#FF9800', // Orange
    critical: '#F44336', // Red
    charging: '#2196F3' // Blue
  },

  animations: {
    lowBattery: {
      enabled: true,
      threshold: 20,
      animation: 'Gentle pulse',
      frequency: 3000 // ms
    },
    criticalBattery: {
      enabled: true,
      threshold: 10,
      animation: 'Faster pulse',
      frequency: 1500
    }
  }
};
```

### Battery Warnings

```typescript
const BATTERY_WARNINGS = {
  lowBatteryWarning: {
    threshold: 20,
    notification: {
      type: 'Toast notification',
      message: 'Battery low (20%). Consider enabling Battery Saver.',
      duration: 5000,
      action: {
        label: 'Enable',
        action: 'enableBatterySaver()'
      },
      dismissable: true
    },
    showOnce: true, // Per session
    respectDoNotDisturb: true
  },

  criticalBatteryWarning: {
    threshold: 10,
    notification: {
      type: 'Modal dialog',
      title: 'Critical Battery',
      message: 'Battery at 10%. Game may close unexpectedly. Save progress by finishing current match or returning to lobby.',
      options: [
        { label: 'Enable Battery Saver', primary: true },
        { label: 'Continue Playing' }
      ]
    },
    showOnce: true
  },

  chargingNotification: {
    showWhen: 'batterySaverEnabled && deviceCharging',
    message: 'Device is charging. Disable Battery Saver for better graphics?',
    autoDisable: false // Require user confirmation
  }
};
```

---

## Background Process Management

### Process Optimization

```typescript
const BACKGROUND_PROCESS_MANAGEMENT = {
  whenOnBattery: {
    analytics: {
      normal: 'Real-time uploads',
      optimized: 'Batch uploads every 5 minutes',
      savings: 'Reduced network radio usage'
    },

    telemetry: {
      normal: 'Continuous collection',
      optimized: 'Reduced sampling rate',
      samplingReduction: '50%'
    },

    assetPreloading: {
      normal: 'Aggressive preloading',
      optimized: 'On-demand loading only',
      affectsLoadTimes: 'Slightly longer level transitions'
    },

    audioStreaming: {
      normal: 'High quality streaming',
      optimized: 'Compressed audio, smaller buffers',
      qualityReduction: 'Minimal perceptible difference'
    },

    networkPolling: {
      normal: 'Frequent server pings',
      optimized: 'Reduced polling frequency',
      interval: {
        normal: 1000, // ms
        optimized: 3000
      }
    }
  },

  whenInBackground: {
    description: 'When app is backgrounded (home button pressed)',
    behavior: {
      pauseRendering: true,
      pauseAudio: true,
      maintainConnection: true, // Keep server connection for rejoin
      connectionTimeout: 120000, // 2 minutes before disconnect
      reduceUpdateRate: '1 update per 5 seconds'
    }
  },

  whenScreenOff: {
    description: 'Device screen turned off during game',
    behavior: {
      pauseCompletely: true,
      showReconnectOnWake: true,
      autoDisconnectAfter: 60000 // 1 minute
    }
  }
};
```

---

## Thermal Management

### Heat Monitoring

```typescript
const THERMAL_MONITORING = {
  description: 'Monitor device temperature and throttle to prevent overheating',

  detection: {
    iOS: {
      api: 'ProcessInfo.thermalState',
      states: ['nominal', 'fair', 'serious', 'critical']
    },
    android: {
      api: 'PowerManager thermal APIs (API 29+)',
      fallback: 'Heuristic based on frame time spikes'
    }
  },

  thresholds: {
    nominal: {
      description: 'Normal operation',
      action: 'None'
    },
    fair: {
      description: 'Slightly warm',
      action: 'Monitor closely, no changes'
    },
    serious: {
      description: 'Hot, throttling recommended',
      action: 'Auto-enable light throttling',
      changes: {
        frameRate: 45, // Reduce from 60
        resolution: '85%',
        particles: '75%'
      }
    },
    critical: {
      description: 'Very hot, immediate action needed',
      action: 'Aggressive throttling',
      changes: {
        frameRate: 30,
        resolution: '70%',
        shadows: 'disabled',
        postProcess: 'disabled',
        particles: '50%'
      },
      notification: {
        show: true,
        message: 'Device is hot. Reducing performance to cool down.',
        type: 'toast'
      }
    }
  },

  recovery: {
    description: 'Restore settings when temperature drops',
    hysteresis: 30000, // Wait 30 seconds after cooling
    gradualRestore: true,
    restoreSteps: 3 // Restore in stages
  }
};
```

### Thermal Notification UI

```typescript
const THERMAL_NOTIFICATION = {
  indicator: {
    showWhen: 'thermalState >= serious',
    icon: 'thermometer_hot',
    position: 'Near battery indicator',
    color: {
      serious: '#FF9800', // Orange
      critical: '#F44336' // Red
    },
    tooltip: 'Device is running hot. Performance reduced.'
  },

  fullWarning: {
    showWhen: 'thermalState === critical',
    type: 'Banner at top of screen',
    message: 'Device overheating - Performance reduced',
    background: '#FF5722',
    autoDismiss: false,
    dismissWhen: 'thermalState < serious'
  },

  recommendations: {
    show: true,
    tips: [
      'Remove phone case to improve cooling',
      'Move to a cooler environment',
      'Take a short break to let device cool',
      'Avoid direct sunlight on device'
    ]
  }
};
```

---

## Settings UI

### Battery & Performance Settings

```typescript
const BATTERY_SETTINGS_UI = {
  location: 'Settings > Performance > Battery',

  sections: {
    batterySaver: {
      title: 'Battery Saver Mode',
      controls: [
        {
          type: 'toggle',
          id: 'battery_saver_enabled',
          label: 'Enable Battery Saver',
          default: false,
          description: 'Reduce graphics for longer play sessions'
        },
        {
          type: 'toggle',
          id: 'battery_saver_auto',
          label: 'Auto-Enable at Low Battery',
          default: true,
          showWhen: '!battery_saver_enabled'
        },
        {
          type: 'slider',
          id: 'battery_saver_threshold',
          label: 'Auto-Enable Threshold',
          range: [10, 30],
          default: 20,
          step: 5,
          unit: '%',
          showWhen: 'battery_saver_auto && !battery_saver_enabled'
        }
      ]
    },

    display: {
      title: 'Battery Display',
      controls: [
        {
          type: 'toggle',
          id: 'show_battery_level',
          label: 'Show Battery in HUD',
          default: false,
          description: 'Display battery percentage during gameplay'
        },
        {
          type: 'toggle',
          id: 'low_battery_warnings',
          label: 'Low Battery Warnings',
          default: true,
          description: 'Notify when battery is running low'
        }
      ]
    },

    thermal: {
      title: 'Thermal Management',
      controls: [
        {
          type: 'toggle',
          id: 'thermal_throttling',
          label: 'Auto-Throttle When Hot',
          default: true,
          description: 'Automatically reduce performance if device overheats'
        },
        {
          type: 'toggle',
          id: 'show_thermal_indicator',
          label: 'Show Temperature Warning',
          default: true,
          description: 'Display indicator when device is hot'
        }
      ]
    },

    advanced: {
      title: 'Advanced',
      controls: [
        {
          type: 'toggle',
          id: 'reduce_background_activity',
          label: 'Reduce Background Activity',
          default: true,
          description: 'Optimize background processes for battery life'
        },
        {
          type: 'info',
          id: 'battery_status',
          label: 'Current Battery',
          value: '{batteryLevel}% {chargingStatus}',
          refreshInterval: 60000
        }
      ]
    }
  },

  previewPanel: {
    show: true,
    content: {
      currentPowerUsage: 'Estimated power draw',
      batterySaverImpact: 'Estimated battery life extension',
      graphicsComparison: 'Before/after visual preview'
    }
  }
};
```

---

## Implementation Details

### Battery State Detection

```typescript
const BATTERY_DETECTION = {
  iOS: {
    api: 'UIDevice.current.batteryLevel',
    monitoring: 'UIDevice.current.isBatteryMonitoringEnabled = true',
    notifications: [
      'UIDevice.batteryLevelDidChangeNotification',
      'UIDevice.batteryStateDidChangeNotification'
    ],
    states: ['unknown', 'unplugged', 'charging', 'full']
  },

  android: {
    api: 'BatteryManager',
    intentFilter: 'Intent.ACTION_BATTERY_CHANGED',
    properties: [
      'BATTERY_PROPERTY_CAPACITY',
      'BATTERY_PROPERTY_STATUS',
      'BATTERY_PROPERTY_CHARGE_COUNTER'
    ]
  },

  polling: {
    interval: 60000, // Check every minute
    onLevelChange: 'Update UI immediately'
  },

  unity: {
    api: 'SystemInfo.batteryLevel', // Returns 0-1
    status: 'SystemInfo.batteryStatus'
  }
};
```

### Quality Preset Application

```typescript
const QUALITY_PRESET_APPLICATION = {
  batterySaverPreset: {
    name: 'Battery Saver',
    settings: {
      targetFrameRate: 30,
      renderScale: 0.7,
      shadowQuality: 0, // Disabled
      shadowDistance: 0,
      antiAliasing: 0, // Disabled
      bloomEnabled: false,
      ambientOcclusionEnabled: false,
      particleDensity: 0.5,
      lodBias: 1.5,
      textureQuality: 1, // Medium
      anisotropicFiltering: 0
    }
  },

  application: {
    method: 'QualitySettings override',
    transitionTime: 500, // ms fade between quality levels
    preserveUserOverrides: false, // Battery saver takes priority
    revertOnDisable: true
  },

  exclusions: {
    description: 'Settings that should not be reduced',
    list: [
      'UI render quality',
      'Text rendering',
      'Critical gameplay indicators'
    ]
  }
};
```

---

## Performance Metrics

### Battery Impact Tracking

```typescript
const BATTERY_METRICS = {
  tracking: {
    sessionBatteryDrain: {
      description: 'Battery % used during session',
      calculation: 'startBattery - endBattery',
      reportFrequency: 'End of session'
    },
    drainRatePerHour: {
      description: 'Estimated battery drain per hour',
      calculation: 'batteryUsed / sessionHours',
      unit: '%/hour'
    },
    batterySaverEffectiveness: {
      description: 'Drain comparison with/without battery saver',
      calculation: 'Compare sessions with similar play patterns'
    }
  },

  targets: {
    normalMode: {
      drainRate: '15-25% per hour', // Typical for mobile games
      acceptable: true
    },
    batterySaverMode: {
      drainRate: '8-15% per hour',
      target: '40% reduction from normal'
    }
  },

  analytics: {
    events: [
      {
        name: 'battery_saver_activated',
        properties: {
          trigger: 'manual | auto',
          battery_level: 'number',
          thermal_state: 'string'
        }
      },
      {
        name: 'thermal_throttling_activated',
        properties: {
          thermal_state: 'string',
          throttle_level: 'light | aggressive'
        }
      },
      {
        name: 'session_battery_usage',
        properties: {
          battery_start: 'number',
          battery_end: 'number',
          session_duration: 'minutes',
          battery_saver_time: 'minutes',
          device_model: 'string'
        }
      }
    ]
  }
};
```

---

## Testing Requirements

### Test Cases

```typescript
const BATTERY_TESTING = {
  functionalTests: [
    {
      name: 'Manual battery saver toggle',
      steps: ['Enable battery saver', 'Verify graphics reduce', 'Disable', 'Verify restore'],
      expected: 'Smooth transition between modes'
    },
    {
      name: 'Auto-activation at threshold',
      steps: ['Set threshold to 20%', 'Drain battery below 20%', 'Observe prompt'],
      expected: 'Prompt appears, accepting enables battery saver'
    },
    {
      name: 'Battery indicator accuracy',
      steps: ['Enable battery display', 'Compare to system battery'],
      expected: 'Matches within 1%'
    },
    {
      name: 'Thermal throttling',
      steps: ['Run intensive benchmark', 'Monitor thermal state', 'Verify throttling'],
      expected: 'Performance reduces when device heats up'
    }
  ],

  performanceTests: [
    {
      name: 'Battery drain comparison',
      method: 'Play 1 hour normal, 1 hour battery saver',
      expected: '30-50% less drain in battery saver'
    },
    {
      name: 'Frame rate verification',
      method: 'Measure FPS in battery saver mode',
      expected: 'Stable 30 FPS with minimal drops'
    },
    {
      name: 'Thermal recovery time',
      method: 'Heat device, measure time to return to normal',
      expected: 'Recovery within 2-5 minutes'
    }
  ],

  deviceMatrix: [
    'iPhone 12/13/14 series',
    'iPhone SE (low-end)',
    'Samsung Galaxy S21/S22/S23',
    'Samsung Galaxy A series (mid-range)',
    'Pixel 6/7 series',
    'Various Android tablets'
  ]
};
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-06 | Development Team | Initial battery optimization specification |
