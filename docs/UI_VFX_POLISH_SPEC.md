# UI VFX and Polish Specification

## Document Information
- **Task ID:** ART-016
- **Priority:** P1
- **Complexity:** S
- **Last Updated:** 2025-12-05
- **Status:** Complete

---

## 1. Overview

### 1.1 Purpose
This specification defines visual effects and polish elements for all UI interactions in Plunderstorm Mobile. These effects enhance the feel of responsiveness, create excitement during reward moments, and reinforce the pirate theme throughout the user interface.

### 1.2 Design Goals
- **Responsive Feel:** Every interaction should have immediate visual feedback
- **Excitement Amplification:** Reward moments should feel celebratory and exciting
- **Theme Consistency:** All effects align with the pirate fantasy aesthetic
- **Performance Safety:** Effects must not impact UI responsiveness
- **Accessibility:** Effects enhance but don't obstruct information

### 1.3 Art Style Alignment
UI VFX follows the established visual style:
- Bold, saturated colors with gold and ocean blue accents
- Stylized particles with pirate motifs (coins, stars, waves)
- Whimsical, slightly exaggerated animations
- Mobile-optimized with clear, readable effects

---

## 2. Button and Interaction Effects

### 2.1 Button Press Effects

```typescript
const BUTTON_PRESS_VFX = {
  primaryButton: {
    states: {
      idle: {
        glow: false,
        scale: 1.0
      },
      hover: {
        glow: true,
        glowColor: '#FFD700',
        glowIntensity: 0.3,
        scale: 1.02,
        transitionDuration: 0.1 // Seconds
      },
      pressed: {
        glow: true,
        glowColor: '#FFA500',
        glowIntensity: 0.5,
        scale: 0.95,
        transitionDuration: 0.05,
        particles: {
          enabled: true,
          type: 'radial_burst',
          count: 6,
          color: '#FFD700',
          size: 4,
          lifetime: 0.3
        }
      },
      released: {
        ripple: {
          enabled: true,
          color: '#FFFFFF',
          opacity: 0.4,
          duration: 0.25,
          expansion: 1.5 // Scale multiplier
        }
      }
    }
  },

  secondaryButton: {
    states: {
      pressed: {
        scale: 0.96,
        transitionDuration: 0.05,
        particles: {
          enabled: false // Simpler for secondary
        }
      }
    }
  },

  iconButton: {
    states: {
      pressed: {
        scale: 0.9,
        rotation: 5, // Degrees wobble
        transitionDuration: 0.08
      }
    }
  }
};
```

### 2.2 Toggle and Slider Effects

```typescript
const TOGGLE_SLIDER_VFX = {
  toggle: {
    onActivate: {
      flash: {
        color: '#4CAF50', // Green
        duration: 0.2,
        intensity: 0.6
      },
      particles: {
        type: 'checkmark_sparkle',
        count: 4,
        color: '#FFFFFF'
      }
    },
    onDeactivate: {
      flash: {
        color: '#9E9E9E', // Grey
        duration: 0.15,
        intensity: 0.3
      }
    }
  },

  slider: {
    onDrag: {
      trailEffect: {
        enabled: true,
        color: '#FFD700',
        opacity: 0.5,
        fadeTime: 0.3
      }
    },
    onRelease: {
      bounce: {
        enabled: true,
        amplitude: 3, // Pixels
        duration: 0.2
      }
    }
  },

  dropdown: {
    onOpen: {
      animation: 'unfold',
      duration: 0.2,
      easing: 'ease_out_back'
    },
    onSelect: {
      flash: {
        color: '#FFD700',
        duration: 0.15
      }
    }
  }
};
```

### 2.3 Tab and Navigation Effects

```typescript
const NAVIGATION_VFX = {
  tabSwitch: {
    indicator: {
      type: 'sliding_underline',
      color: '#FFD700',
      thickness: 3,
      animationDuration: 0.25,
      easing: 'ease_out_cubic'
    },
    activeTab: {
      glow: true,
      glowColor: '#FFD700',
      glowIntensity: 0.2
    }
  },

  screenTransition: {
    type: 'slide_fade',
    duration: 0.3,
    overlayParticles: {
      enabled: true,
      type: 'gold_dust',
      count: 20,
      opacity: 0.3
    }
  },

  backButton: {
    onPress: {
      swoosh: {
        enabled: true,
        direction: 'left',
        color: '#FFFFFF',
        opacity: 0.4
      }
    }
  }
};
```

---

## 3. Reward Reveal Effects

### 3.1 Chest Opening Sequence

```typescript
const CHEST_OPENING_VFX = {
  // Full reward chest opening ceremony
  sequence: {
    phases: [
      {
        name: 'anticipation',
        duration: 0.5,
        effects: {
          chestShake: {
            amplitude: 3,
            frequency: 10,
            intensity: 'building'
          },
          glowBuildup: {
            color: '#FFD700',
            startIntensity: 0.1,
            endIntensity: 0.8
          },
          particles: {
            type: 'rising_sparkles',
            count: 15,
            color: ['#FFD700', '#FFFFFF']
          }
        }
      },
      {
        name: 'burst',
        duration: 0.3,
        effects: {
          chestOpen: {
            animation: 'lid_flip',
            duration: 0.2
          },
          lightBurst: {
            type: 'radial',
            color: '#FFFFFF',
            intensity: 1.0,
            fadeTime: 0.5
          },
          confetti: {
            enabled: true,
            count: 50,
            colors: ['#FFD700', '#FF6B35', '#4169E1', '#32CD32'],
            spread: 180,
            velocity: 400
          },
          coinShower: {
            enabled: true,
            count: 20,
            texture: 'gold_coin',
            velocity: 300,
            gravity: 500
          }
        }
      },
      {
        name: 'reveal',
        duration: 0.8,
        effects: {
          itemRise: {
            animation: 'float_up',
            height: 100, // Pixels
            bobbing: true
          },
          itemGlow: {
            matchRarity: true, // Color based on item rarity
            intensity: 0.6,
            pulse: true
          },
          backgroundDim: {
            opacity: 0.7,
            duration: 0.3
          }
        }
      }
    ]
  },

  // Rarity-specific enhancements
  rarityEnhancements: {
    common: {
      glowColor: '#9E9E9E',
      confettiCount: 20,
      coinCount: 5,
      soundIntensity: 'light'
    },
    uncommon: {
      glowColor: '#4CAF50',
      confettiCount: 30,
      coinCount: 10,
      soundIntensity: 'medium'
    },
    rare: {
      glowColor: '#2196F3',
      confettiCount: 40,
      coinCount: 15,
      lightningFlash: true,
      soundIntensity: 'high'
    },
    epic: {
      glowColor: '#9C27B0',
      confettiCount: 50,
      coinCount: 20,
      lightningFlash: true,
      screenShake: { amplitude: 5, duration: 0.2 },
      soundIntensity: 'epic'
    },
    legendary: {
      glowColor: '#FF9800',
      confettiCount: 75,
      coinCount: 30,
      lightningFlash: true,
      screenShake: { amplitude: 8, duration: 0.3 },
      beamOfLight: true,
      chorusEffect: true,
      soundIntensity: 'legendary'
    }
  }
};
```

### 3.2 Confetti System

```typescript
const CONFETTI_SYSTEM = {
  // Reusable confetti for various celebrations
  presets: {
    light: {
      count: 20,
      colors: ['#FFD700', '#FFFFFF'],
      spread: 90,
      velocity: 200,
      gravity: 300,
      lifetime: 2.0,
      rotationSpeed: 180 // Degrees per second
    },
    medium: {
      count: 40,
      colors: ['#FFD700', '#FF6B35', '#4169E1', '#FFFFFF'],
      spread: 120,
      velocity: 300,
      gravity: 400,
      lifetime: 2.5,
      rotationSpeed: 270
    },
    heavy: {
      count: 75,
      colors: ['#FFD700', '#FF6B35', '#4169E1', '#32CD32', '#FF69B4', '#FFFFFF'],
      spread: 180,
      velocity: 400,
      gravity: 500,
      lifetime: 3.0,
      rotationSpeed: 360
    }
  },

  particleProperties: {
    shapes: ['rectangle', 'circle', 'star'],
    sizeRange: { min: 5, max: 12 },
    opacityFade: {
      startFade: 0.7, // Start fading at 70% lifetime
      endOpacity: 0
    },
    flutter: {
      enabled: true,
      amplitude: 20,
      frequency: 3
    }
  },

  pirateThemed: {
    enabled: true,
    specialParticles: [
      { shape: 'skull', probability: 0.05 },
      { shape: 'anchor', probability: 0.05 },
      { shape: 'coin', probability: 0.1 }
    ]
  }
};
```

### 3.3 Currency Gain/Spend Effects

```typescript
const CURRENCY_VFX = {
  // Gold (soft currency)
  gold: {
    gain: {
      coinFly: {
        enabled: true,
        fromPosition: 'source', // Loot, quest, etc.
        toPosition: 'currency_display',
        count: 5, // Fly in groups
        duration: 0.6,
        curve: 'arc',
        trailEffect: {
          enabled: true,
          color: '#FFD700',
          opacity: 0.5
        }
      },
      counterRoll: {
        enabled: true,
        duration: 0.4,
        overshoot: true // Goes slightly past then settles
      },
      flash: {
        color: '#FFD700',
        duration: 0.2
      },
      particles: {
        type: 'sparkle_burst',
        count: 8,
        color: '#FFD700'
      }
    },
    spend: {
      coinFly: {
        enabled: true,
        fromPosition: 'currency_display',
        toPosition: 'target', // Purchase button, etc.
        count: 3,
        duration: 0.4
      },
      counterRoll: {
        enabled: true,
        duration: 0.3,
        color: '#FF4444' // Red during decrease
      },
      shake: {
        amplitude: 2,
        duration: 0.1
      }
    }
  },

  // Doubloons (premium currency)
  doubloons: {
    gain: {
      coinFly: {
        enabled: true,
        count: 8,
        duration: 0.8,
        trailEffect: {
          enabled: true,
          color: '#E040FB',
          opacity: 0.6,
          glow: true
        }
      },
      flash: {
        color: '#E040FB',
        duration: 0.3,
        intensity: 0.8
      },
      particles: {
        type: 'premium_sparkle',
        count: 15,
        colors: ['#E040FB', '#FFFFFF', '#FFD700']
      },
      shimmer: {
        enabled: true,
        duration: 1.0
      }
    },
    spend: {
      coinFly: {
        enabled: true,
        count: 5,
        duration: 0.5
      },
      counterRoll: {
        enabled: true,
        duration: 0.4
      }
    }
  },

  // XP gain
  xp: {
    gain: {
      orbs: {
        enabled: true,
        count: 3,
        color: '#00E5FF',
        size: 8,
        duration: 0.5
      },
      barFill: {
        animation: 'smooth_fill',
        duration: 0.6,
        glow: true,
        glowColor: '#00E5FF'
      },
      particles: {
        type: 'rising_sparkles',
        count: 10,
        color: '#00E5FF'
      }
    }
  }
};
```

---

## 4. Level Up Celebration

### 4.1 Level Up Sequence

```typescript
const LEVEL_UP_VFX = {
  sequence: {
    phases: [
      {
        name: 'buildup',
        duration: 0.3,
        effects: {
          xpBarComplete: {
            flash: true,
            color: '#00E5FF',
            intensity: 1.0
          },
          screenPulse: {
            color: '#FFFFFF',
            opacity: 0.3
          }
        }
      },
      {
        name: 'burst',
        duration: 0.4,
        effects: {
          levelBadge: {
            animation: 'scale_bounce',
            from: 0.5,
            to: 1.2,
            settle: 1.0,
            duration: 0.4
          },
          radialBurst: {
            color: '#FFD700',
            rings: 3,
            expansion: 2.0
          },
          confetti: 'medium',
          screenFlash: {
            color: '#FFD700',
            intensity: 0.4,
            duration: 0.1
          }
        }
      },
      {
        name: 'celebration',
        duration: 1.5,
        effects: {
          levelNumber: {
            glow: true,
            glowColor: '#FFD700',
            pulse: true,
            pulseSpeed: 2.0
          },
          floatingText: {
            text: 'LEVEL UP!',
            color: '#FFD700',
            fontSize: 48,
            animation: 'float_fade',
            duration: 1.5
          },
          ribbons: {
            enabled: true,
            count: 4,
            colors: ['#FFD700', '#FF6B35']
          }
        }
      }
    ]
  },

  // Milestone levels (10, 25, 50, etc.)
  milestoneEnhancements: {
    extraConfetti: true,
    fireworks: {
      enabled: true,
      count: 5,
      colors: ['#FFD700', '#FF6B35', '#4169E1']
    },
    specialBadge: {
      animation: 'spin_in',
      shimmer: true
    }
  }
};
```

### 4.2 XP Bar Animations

```typescript
const XP_BAR_VFX = {
  // Continuous XP gain feedback
  filling: {
    animation: 'smooth_fill',
    glowTrail: {
      enabled: true,
      color: '#00E5FF',
      width: 4
    },
    particles: {
      enabled: true,
      type: 'leading_sparkles',
      count: 3,
      color: '#FFFFFF'
    }
  },

  // Near level up
  almostFull: {
    threshold: 0.9, // 90% full
    glow: {
      enabled: true,
      color: '#00E5FF',
      pulse: true,
      intensity: 0.5
    },
    particles: {
      type: 'anticipation_sparkles',
      continuous: true
    }
  },

  // Overflow animation (excess XP to next level)
  overflow: {
    wrapAnimation: 'flash_reset',
    excessVisualization: 'quick_fill'
  }
};
```

---

## 5. Achievement Unlock Effects

### 5.1 Achievement Popup

```typescript
const ACHIEVEMENT_VFX = {
  popup: {
    entrance: {
      animation: 'slide_in_bounce',
      from: 'top',
      duration: 0.4,
      easing: 'ease_out_back'
    },
    display: {
      duration: 3.0, // Time shown
      shimmer: {
        enabled: true,
        color: '#FFD700',
        interval: 0.5
      }
    },
    exit: {
      animation: 'slide_out_fade',
      direction: 'top',
      duration: 0.3
    }
  },

  iconReveal: {
    animation: 'stamp_in',
    scale: { from: 2.0, to: 1.0 },
    rotation: { from: -15, to: 0 },
    duration: 0.3,
    impactParticles: {
      enabled: true,
      type: 'radial_burst',
      count: 12,
      color: '#FFD700'
    }
  },

  backgroundEffects: {
    glow: {
      color: '#FFD700',
      intensity: 0.3
    },
    particles: {
      type: 'floating_sparkles',
      count: 8,
      continuous: true
    }
  },

  // Tier-based enhancements
  tierEffects: {
    bronze: {
      glowColor: '#CD7F32',
      particleCount: 6
    },
    silver: {
      glowColor: '#C0C0C0',
      particleCount: 10
    },
    gold: {
      glowColor: '#FFD700',
      particleCount: 15,
      confetti: 'light'
    }
  }
};
```

### 5.2 Achievement List Effects

```typescript
const ACHIEVEMENT_LIST_VFX = {
  // When viewing completed achievements
  completedBadge: {
    glow: true,
    glowColor: '#FFD700',
    shimmer: {
      enabled: true,
      interval: 3.0
    }
  },

  // Progress bar animation
  progressUpdate: {
    fillAnimation: 'smooth',
    duration: 0.5,
    particles: {
      onMilestone: true, // 25%, 50%, 75%
      type: 'mini_burst',
      count: 5
    }
  },

  // Newly unlocked indicator
  newIndicator: {
    badge: {
      text: 'NEW',
      color: '#FF4444',
      pulse: true
    },
    highlight: {
      color: '#FFD700',
      opacity: 0.2
    }
  }
};
```

---

## 6. Battle Pass Effects

### 6.1 Tier Unlock Celebration

```typescript
const BATTLE_PASS_VFX = {
  tierUnlock: {
    // Standard tier unlock
    standard: {
      barProgress: {
        fillAnimation: 'glow_sweep',
        duration: 0.4
      },
      tierBadge: {
        animation: 'pop_in',
        scale: { from: 0, to: 1.1, settle: 1.0 },
        duration: 0.3
      },
      rewardReveal: {
        animation: 'flip_in',
        duration: 0.4,
        glow: true
      },
      particles: {
        type: 'celebration_burst',
        count: 15,
        colors: ['#FFD700', '#FFFFFF']
      }
    },

    // Premium tier unlock
    premium: {
      barProgress: {
        fillAnimation: 'premium_glow_sweep',
        color: '#E040FB',
        duration: 0.5
      },
      tierBadge: {
        animation: 'spin_pop',
        shimmer: true
      },
      rewardReveal: {
        animation: 'dramatic_reveal',
        duration: 0.6,
        lightBeam: true
      },
      particles: {
        type: 'premium_sparkle',
        count: 25,
        colors: ['#E040FB', '#FFD700', '#FFFFFF']
      },
      confetti: 'medium'
    },

    // Milestone tiers (10, 25, 50, etc.)
    milestone: {
      screenEffect: {
        flash: true,
        shake: { amplitude: 5, duration: 0.2 }
      },
      fireworks: {
        enabled: true,
        count: 3
      },
      specialAnimation: 'grand_reveal'
    }
  },

  // Battle Pass track visualization
  trackEffects: {
    currentPosition: {
      marker: {
        glow: true,
        glowColor: '#00E5FF',
        bounce: true
      }
    },
    unlockedTiers: {
      glow: {
        color: '#FFD700',
        intensity: 0.2
      }
    },
    lockedTiers: {
      style: 'dimmed',
      opacity: 0.5
    }
  }
};
```

### 6.2 Battle Pass Purchase Effects

```typescript
const BATTLE_PASS_PURCHASE_VFX = {
  // Initial purchase celebration
  purchaseCelebration: {
    sequence: [
      {
        name: 'transition',
        effect: 'screen_wipe',
        color: '#E040FB',
        duration: 0.3
      },
      {
        name: 'reveal',
        effects: {
          titleAnimation: {
            text: 'PREMIUM UNLOCKED!',
            animation: 'slam_in',
            color: '#FFD700'
          },
          lightBurst: {
            color: '#FFFFFF',
            intensity: 0.8
          },
          confetti: 'heavy'
        },
        duration: 1.0
      },
      {
        name: 'rewards_preview',
        effects: {
          rewardsCarousel: {
            animation: 'fly_in_sequence',
            delay: 0.1 // Between items
          }
        },
        duration: 2.0
      }
    ]
  },

  // Track transformation
  trackTransform: {
    animation: 'shimmer_reveal',
    premiumHighlight: {
      color: '#E040FB',
      glow: true
    }
  }
};
```

---

## 7. Shop and Purchase Effects

### 7.1 Purchase Confirmation

```typescript
const PURCHASE_VFX = {
  confirmation: {
    success: {
      checkmark: {
        animation: 'draw_in',
        color: '#4CAF50',
        duration: 0.4
      },
      pulse: {
        color: '#4CAF50',
        rings: 2
      },
      particles: {
        type: 'success_sparkles',
        count: 12,
        colors: ['#4CAF50', '#FFFFFF']
      },
      itemAnimation: {
        type: 'fly_to_inventory',
        duration: 0.6,
        trail: true
      }
    },

    pending: {
      spinner: {
        color: '#FFD700',
        style: 'coin_flip'
      },
      shimmer: {
        continuous: true
      }
    }
  },

  // Item preview in shop
  itemPreview: {
    hover: {
      scale: 1.05,
      glow: true,
      glowColor: 'rarity_based'
    },
    select: {
      scale: 1.1,
      spotlight: {
        enabled: true,
        color: '#FFFFFF',
        opacity: 0.3
      }
    }
  }
};
```

### 7.2 Special Offer Effects

```typescript
const SPECIAL_OFFER_VFX = {
  limitedTime: {
    banner: {
      animation: 'shimmer_loop',
      color: '#FF4444',
      interval: 2.0
    },
    timer: {
      urgency: {
        threshold: 3600, // Last hour
        effects: {
          pulse: true,
          color: '#FF4444'
        }
      }
    }
  },

  discount: {
    badge: {
      animation: 'bounce_attention',
      interval: 5.0
    },
    priceSlash: {
      animation: 'strike_through',
      color: '#FF4444'
    }
  },

  featured: {
    spotlight: {
      enabled: true,
      color: '#FFD700',
      rays: true
    },
    particles: {
      type: 'floating_gems',
      count: 5,
      continuous: true
    }
  }
};
```

---

## 8. Match Result Effects

### 8.1 Victory Celebration

```typescript
const VICTORY_VFX = {
  sequence: [
    {
      name: 'announcement',
      duration: 1.0,
      effects: {
        title: {
          text: 'VICTORY!',
          animation: 'slam_grow',
          color: '#FFD700',
          glow: true
        },
        screenFlash: {
          color: '#FFD700',
          intensity: 0.5
        }
      }
    },
    {
      name: 'celebration',
      duration: 3.0,
      effects: {
        confetti: 'heavy',
        fireworks: {
          enabled: true,
          count: 8,
          continuous: true,
          interval: 0.5
        },
        ribbons: {
          enabled: true,
          count: 6,
          colors: ['#FFD700', '#FF6B35']
        },
        coinShower: {
          enabled: true,
          count: 30,
          duration: 2.0
        }
      }
    },
    {
      name: 'rewards',
      duration: 2.0,
      effects: {
        xpGain: {
          animation: 'dramatic_count',
          particles: true
        },
        currencyGain: {
          animation: 'coin_fly_burst'
        }
      }
    }
  ],

  // Placement-based variations
  placements: {
    first: 'full_celebration',
    second: 'silver_celebration',
    third: 'bronze_celebration',
    top10: 'good_result'
  }
};
```

### 8.2 Defeat Screen Effects

```typescript
const DEFEAT_VFX = {
  // Tasteful, encouraging defeat effects
  sequence: [
    {
      name: 'announcement',
      duration: 0.5,
      effects: {
        title: {
          text: 'DEFEATED',
          animation: 'fade_in',
          color: '#9E9E9E'
        },
        vignette: {
          color: '#000000',
          intensity: 0.3
        }
      }
    },
    {
      name: 'stats',
      duration: 1.5,
      effects: {
        placement: {
          animation: 'count_down',
          highlight: 'placement_tier'
        },
        kills: {
          animation: 'count_up',
          particles: {
            enabled: true,
            count: 3,
            color: '#FF4444'
          }
        }
      }
    },
    {
      name: 'encouragement',
      effects: {
        xpGain: {
          animation: 'smooth_count',
          message: 'XP Earned!'
        },
        motivationalText: {
          random: ['Better luck next time!', 'Almost had it!', 'Keep sailing!'],
          animation: 'fade_in'
        }
      }
    }
  ]
};
```

---

## 9. Notification Effects

### 9.1 Toast Notifications

```typescript
const TOAST_VFX = {
  types: {
    info: {
      icon: 'info_circle',
      color: '#2196F3',
      entrance: 'slide_in_right',
      exit: 'fade_out',
      duration: 3.0
    },
    success: {
      icon: 'checkmark',
      color: '#4CAF50',
      entrance: 'pop_in',
      particles: {
        enabled: true,
        type: 'success_sparkle',
        count: 4
      },
      exit: 'slide_out_right',
      duration: 2.5
    },
    warning: {
      icon: 'warning_triangle',
      color: '#FF9800',
      entrance: 'shake_in',
      pulse: true,
      exit: 'fade_out',
      duration: 4.0
    },
    error: {
      icon: 'error_x',
      color: '#F44336',
      entrance: 'shake_in',
      shake: { amplitude: 5, duration: 0.3 },
      exit: 'fade_out',
      duration: 5.0
    },
    reward: {
      icon: 'gift',
      color: '#FFD700',
      entrance: 'bounce_in',
      glow: true,
      particles: {
        enabled: true,
        type: 'gold_sparkle',
        count: 8
      },
      exit: 'float_up_fade',
      duration: 4.0
    }
  },

  stacking: {
    maxVisible: 3,
    offset: 10, // Pixels between
    animation: 'push_down'
  }
};
```

### 9.2 Badge Notifications

```typescript
const BADGE_NOTIFICATION_VFX = {
  newBadge: {
    appearance: {
      animation: 'pop_bounce',
      scale: { from: 0, to: 1.2, settle: 1.0 }
    },
    attention: {
      pulse: true,
      pulseColor: '#FF4444',
      interval: 2.0
    }
  },

  countChange: {
    increment: {
      animation: 'count_up_bounce',
      duration: 0.2
    },
    decrement: {
      animation: 'count_down',
      duration: 0.15
    }
  },

  clearAll: {
    animation: 'shrink_fade',
    duration: 0.3
  }
};
```

---

## 10. Loading and Progress Effects

### 10.1 Loading Indicators

```typescript
const LOADING_VFX = {
  // Main loading screen
  fullScreen: {
    background: {
      animation: 'subtle_wave',
      color: '#1A2A3A'
    },
    spinner: {
      type: 'ship_wheel',
      rotationSpeed: 90, // Degrees per second
      color: '#FFD700'
    },
    tips: {
      animation: 'fade_cycle',
      interval: 5.0
    },
    progressBar: {
      fillAnimation: 'smooth',
      glow: true,
      glowColor: '#00E5FF'
    }
  },

  // Inline loading
  inline: {
    spinner: {
      type: 'coin_flip',
      size: 24,
      color: '#FFD700'
    }
  },

  // Button loading state
  buttonLoading: {
    spinner: {
      type: 'dots_pulse',
      count: 3,
      color: '#FFFFFF'
    }
  }
};
```

### 10.2 Progress Bars

```typescript
const PROGRESS_BAR_VFX = {
  standard: {
    fill: {
      animation: 'smooth_slide',
      glow: true
    },
    milestones: {
      markers: true,
      onReach: {
        flash: true,
        particles: true
      }
    }
  },

  download: {
    fill: {
      animation: 'stripe_scroll',
      stripeColor: 'rgba(255,255,255,0.2)'
    },
    percentage: {
      countAnimation: true
    }
  },

  matchmaking: {
    style: 'pulsing_glow',
    color: '#00E5FF',
    indeterminate: true
  }
};
```

---

## 11. Performance Specifications

### 11.1 Performance Budgets

```typescript
const UI_VFX_PERFORMANCE = {
  particleBudgets: {
    maxSimultaneousParticles: 200,
    maxParticleEmitters: 10,
    particlePoolSize: 500
  },

  animationBudgets: {
    maxSimultaneousTweens: 20,
    targetFrameRate: 60,
    maxFrameTimeMs: 2 // For UI VFX only
  },

  memoryBudgets: {
    particleTextures: '2MB',
    animationData: '1MB',
    effectPresets: '0.5MB'
  },

  optimizationStrategies: {
    particlePooling: true,
    batchedRendering: true,
    frustumCulling: false, // UI always visible
    lodSystem: false // UI doesn't need LOD
  }
};
```

### 11.2 Quality Tier Adjustments

```typescript
const UI_VFX_QUALITY_TIERS = {
  low: {
    particleMultiplier: 0.5,
    confettiEnabled: false,
    complexAnimations: false,
    glowEffects: false,
    shimmerEffects: false
  },

  medium: {
    particleMultiplier: 0.75,
    confettiEnabled: true,
    complexAnimations: true,
    glowEffects: true,
    shimmerEffects: false
  },

  high: {
    particleMultiplier: 1.0,
    confettiEnabled: true,
    complexAnimations: true,
    glowEffects: true,
    shimmerEffects: true
  }
};
```

### 11.3 Effect Priorities

```typescript
const EFFECT_PRIORITIES = {
  // Effects that should never be skipped
  critical: [
    'button_press_feedback',
    'purchase_confirmation',
    'error_feedback'
  ],

  // Can be reduced on low-end devices
  important: [
    'reward_reveal',
    'level_up',
    'achievement_unlock',
    'currency_gain'
  ],

  // Can be disabled if needed
  polish: [
    'confetti',
    'fireworks',
    'ambient_particles',
    'shimmer_effects'
  ]
};
```

---

## 12. Implementation Guidelines

### 12.1 Animation Timing Reference

```typescript
const ANIMATION_TIMING = {
  // Standard durations
  instant: 0.05, // Immediate feedback
  fast: 0.15, // Quick transitions
  normal: 0.3, // Standard animations
  slow: 0.5, // Dramatic reveals
  dramatic: 1.0, // Major celebrations

  // Easing functions
  easings: {
    button: 'ease_out_quad',
    popup: 'ease_out_back',
    celebration: 'ease_out_elastic',
    fade: 'ease_in_out_sine',
    bounce: 'ease_out_bounce'
  }
};
```

### 12.2 Color Reference

```typescript
const UI_VFX_COLORS = {
  // Primary
  gold: '#FFD700',
  premiumPurple: '#E040FB',
  xpCyan: '#00E5FF',

  // Feedback
  success: '#4CAF50',
  warning: '#FF9800',
  error: '#F44336',
  info: '#2196F3',

  // Rarity
  common: '#9E9E9E',
  uncommon: '#4CAF50',
  rare: '#2196F3',
  epic: '#9C27B0',
  legendary: '#FF9800',

  // Effects
  sparkle: '#FFFFFF',
  glow: '#FFD700',
  shadow: 'rgba(0,0,0,0.5)'
};
```

---

## 13. Testing Checklist

### 13.1 Functionality Tests

| Test | Expected Result |
|------|-----------------|
| All buttons have press feedback | Visual response within 50ms |
| Chest opening plays full sequence | All phases complete smoothly |
| Currency animations fly correctly | From source to destination |
| Level up celebration triggers | All effects synchronized |
| Confetti spawns and falls | Natural physics, fades properly |
| Notifications stack correctly | Max 3 visible, proper offset |
| Loading indicators animate | Smooth, non-blocking |

### 13.2 Performance Tests

| Test | Pass Criteria |
|------|--------------|
| 200 particles simultaneous | Maintains 60 FPS |
| Heavy confetti celebration | No frame drops below 30 FPS |
| Rapid button pressing | Responsive without lag |
| Extended reward sequence | No memory leaks |
| Low-end device (full quality) | Graceful degradation |

### 13.3 Visual Quality Tests

| Test | Criteria |
|------|----------|
| Art style consistency | Matches pirate theme |
| Effect visibility | Clear on all backgrounds |
| Color consistency | Matches defined palette |
| Animation smoothness | No stuttering or jank |
| Particle appearance | Crisp, not pixelated |

---

## Appendix A: Asset Requirements

### Particle Textures
| Asset | Size | Format | Description |
|-------|------|--------|-------------|
| sparkle_01 | 32x32 | PNG Alpha | Basic sparkle |
| star_particle | 32x32 | PNG Alpha | Star shape |
| coin_particle | 32x32 | PNG Alpha | Small gold coin |
| confetti_sheet | 128x32 | PNG Alpha | 4 confetti shapes |
| smoke_soft | 64x64 | PNG Alpha | Soft smoke puff |
| glow_radial | 64x64 | PNG Alpha | Radial glow |

### Animation Sprites
| Asset | Size | Frames | Description |
|-------|------|--------|-------------|
| checkmark_draw | 64x64 | 8 | Animated checkmark |
| ship_wheel | 64x64 | 1 | Rotating wheel |
| coin_flip | 32x32 | 6 | Flipping coin |

---

## Appendix B: Sound Effect Pairings

Each VFX should be accompanied by appropriate SFX (defined in SOUND_EFFECTS_SPEC.md):

| Effect | Sound |
|--------|-------|
| Button press | ui_button_click |
| Chest open | chest_open_burst |
| Confetti | celebration_pop |
| Coin gain | coin_collect |
| Level up | level_up_fanfare |
| Achievement | achievement_unlock |
| Error | ui_error_buzz |
| Success | ui_success_chime |
