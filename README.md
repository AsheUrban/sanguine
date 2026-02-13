# Sanguine

*Your cycle and your conditions, finally tracked together.*

Sanguine is a privacy-first period and chronic illness tracker that finds the patterns between your menstrual cycle and your health conditions. It's built for people who live with both — and for a world where health data privacy isn't optional.

---

## Why Sanguine Exists

Period trackers treat your cycle in isolation. Chronic illness trackers treat your conditions in isolation. But if you have endometriosis, fibromyalgia, IBS, PMDD, migraines, or any condition that flares cyclically — you already know these things are connected. Your doctor probably knows too. But nobody gives you the data to prove it.

Sanguine sits at that intersection. It tracks your cycle *and* your conditions, finds the correlations, and gives you doctor-ready insights like "your brain fog is 3× more likely during your luteal phase" instead of useless generics like "your pain is higher during your period."

The market gap is clear. Clue and Flo track cycles but miss chronic illness. Bearable tracks everything but overwhelms with data. Nobody filters for signal — nobody says "we know your period hurts, here's what you *don't* already know."

---

## Values

### 1. Zero-knowledge privacy

We don't collect data. Not anonymized data, not metadata, not analytics, not crash logs with device IDs. Zero.

This isn't a policy decision — it's an architectural one. There is no server. There is no account system. There is no database we operate. We cannot access your information under any circumstances, including legal compulsion, because we do not possess it.

This matters because period tracking data has been subpoenaed in the United States. It matters because health data has been shared with advertisers by apps that promised otherwise. And it matters because some of our users are in situations where someone finding this app on their phone could put them at risk.

We will never build infrastructure that contradicts this commitment. If a feature requires a server, we won't build it.

### 2. Safety by design

Privacy-from-corporations and privacy-from-someone-looking-over-your-shoulder are different problems. We design for both.

- **Stealth mode.** Alternative app icon and name. Your home screen doesn't reveal what this app does.
- **App lock.** Biometric or PIN, separate from device lock. Failed attempts can surface a decoy empty state.
- **Discreet notifications.** Vague by default. Never surface health details on a lock screen unless explicitly opted in.
- **Quick exit.** A gesture that instantly drops to a neutral screen.
- **Generic exports.** Backup files are named generically with no recognizable extensions or app references.
- **Encrypted storage.** Data at rest is encrypted on-device, not stored as readable JSON.

These features exist quietly in settings. Nobody has to self-identify as "at risk" to access them. They're just there.

### 3. Respect the user's intelligence

People with chronic conditions are experts in their own bodies. The app should never:

- Tell them something obvious ("your pain is higher during your period")
- Present raw data without interpretation
- Use a generic 1-5 pain scale that turns everything red for someone with fibromyalgia
- Require medical literacy to understand an insight

The insights engine filters for what's *surprising and actionable.* If a pain-type condition peaks during the menstrual phase, we skip it and look for the second-highest correlation instead. We surface "your joint pain clusters in days 22-25 of your luteal phase" because that's what you bring to your doctor.

### 4. Fast check-ins, slow insights

Logging should take under 15 seconds. Tap what's present, tap again to increase severity, done. No sliders, no text fields, no multi-step flows. The value isn't in the act of logging — it's in what emerges over weeks and months.

Insights require patience and we're honest about that. "Keep logging — patterns appear after about a week" is better than inventing correlations from three data points.

### 5. Source-available for trust

Our privacy claim is "we can't see your data." Making the source code public means you don't have to trust us — you can verify. No hidden analytics, no secret endpoints, no telemetry we forgot to mention.

The code is publicly readable but not licensed for reuse. This protects the product while giving users, security researchers, and journalists full ability to audit our privacy claims. Transparency without exploitation.

---

## Design Principles

### Visual identity

- **Dark theme.** `#0e0e11` base, crimson `#e63956` accent. Health apps shouldn't feel clinical — this should feel like something you want to open.
- **Typography.** Cormorant Garamond (serif, italic) for brand and headings. DM Sans for body and UI. The contrast between editorial serif and clean sans creates a warm but precise tone.
- **Severity as color.** Mild `#f0b8c4`, moderate `#d4566a`, severe `#e63956`. No text labels needed — the progression is intuitive. A small colored dot indicates intensity.

### Information architecture

- **Flow first.** This is a period tracker that also does chronic conditions, not the other way around. Flow logging is always at the top.
- **Period symptoms and chronic conditions are separate categories.** Period symptoms (cramps, bloating, breast soreness) are expected to correlate with menstruation. Chronic conditions (migraine, IBS, brain fog) are where surprising patterns live. The check-in separates them visually but uses the same interaction model.
- **Tap-to-cycle severity.** One tap: mild. Two: moderate. Three: severe. Four: off. Same interaction everywhere — check-in, calendar modal, both symptom types. No second screen, no mode switching.
- **Energy and mood are optional.** They default to null, not neutral. No data is better than fake data. Only recorded values are used in analysis.

### Insights

- **Condition-specific, never generic.** Always "your brain fog" or "your back pain," never "your pain."
- **Filtered for non-obvious patterns.** Pain during menstruation is filtered out. Energy dips during menstruation are filtered out. The engine looks for what you *don't* already know.
- **Severity-aware.** "2× more likely during your luteal phase — and tends to be severe" tells you more than frequency alone.
- **Phase-contextual on the home screen.** The insight card shows what's relevant to where you are in your cycle *right now*, not your strongest overall pattern.
- **Swipeable pattern cards.** One insight per card with a cycle ring showing the full cycle proportionally, the relevant phase highlighted. You see *where* in your month this happens spatially.

### Customization

- **Conditions are fully customizable.** Default list of 16 common conditions, plus free-text "add your own" in onboarding, settings, and directly from the check-in screen.
- **Period symptoms are customizable.** Default list of 10 common symptoms, plus add-your-own inline.
- **First-run hints disappear.** "Tap to report · tap again to increase severity" shows the first two check-in visits, then it's gone. The app teaches by doing, not by persisting instructions.

---

## Architecture

### On-device only

```
User's device
├── Encrypted local storage (MMKV)
│   ├── Cycle data (flow, period symptoms with severity)
│   ├── Wellness data (conditions with severity, energy, mood)
│   ├── User preferences (cycle length, selected conditions, settings)
│   └── App state (stealth mode, notification preferences)
├── Platform-native backup (iCloud Keychain / Google Auto Backup)
│   └── Invisible to other apps, encrypted by platform
└── Manual encrypted export (user-initiated)
    └── Generic filename, AES-256, user-set password
```

There is no server component. There is no API. There is no account.

### Data model

```
Period data: {
  "2026-02-10": {
    flow: "heavy" | "medium" | "light",
    symptoms: { cramps: "severe", bloating: "mild" }
  }
}

Wellness data: {
  "2026-02-10": {
    energy: 1-5 | null,
    mood: 1-5 | null,
    flares: { migraine: "severe", brainfog: "moderate" }
  }
}
```

Energy and mood are nullable. If untouched, they don't exist in the record. The insights engine only analyzes days where the user actively logged a value.

### Insights engine

The engine compares per-phase occurrence rates against overall baselines:

1. **Condition insights.** For each tracked condition, calculate frequency and average severity per cycle phase. Compare to overall baseline. Only surface when the difference is meaningful (≥1.3× ratio, minimum 3 occurrences).
2. **Obvious-pattern filter.** If a pain-type condition peaks during the menstrual phase, skip it. Look for the second-highest phase instead and frame as "also spikes during your [phase]."
3. **Energy and mood.** Skip menstrual phase entirely (obvious dip). Only surface follicular/ovulatory/luteal patterns when ≥18% above or below baseline.
4. **Period symptom crossover.** If period symptoms appear meaningfully outside menstruation (e.g., bloating during luteal), surface that as a finding.

### Backup strategy

**Default: platform-native.** iCloud Keychain (iOS) and Auto Backup (Android) handle encrypted sync across the user's own devices without any visible file or user action. This protects the 90% who will never manually back up.

**Optional: encrypted file export.** For cross-platform migration or full user control. AES-256 encryption with a user-set password. Generic filename with no app-identifiable metadata. The user stores it wherever they choose.

**Doctor export: intentional sharing.** A separate "Export report" flow generates a PDF summary of cycle and condition correlations. This is an explicit, conscious act of sharing — the app is transparent that once exported, the data is governed by wherever the user sends it.

---

## Technical Stack

- **Framework:** React Native + Expo (managed workflow)
- **Navigation:** Expo Router (file-based, tab navigation)
- **Storage:** react-native-mmkv (encrypted, synchronous)
- **State management:** Zustand (persisted to MMKV)
- **Animations:** React Native Reanimated + Gesture Handler
- **SVG:** react-native-svg (cycle rings, custom visualizations)
- **PDF export:** expo-print (HTML to PDF, local generation)
- **Notifications:** expo-notifications (local only, no push server)
- **Testing:** Jest + React Native Testing Library, Detox for E2E
- **CI/CD:** EAS Build + EAS Update

### What we don't use

- No analytics SDK (no Mixpanel, no Amplitude, no Firebase Analytics)
- No crash reporting that phones home (no Sentry, no Crashlytics with device IDs)
- No remote config or feature flags that require a server
- No authentication library (no accounts to authenticate)
- No networking library (no API to call)

---

## Privacy Policy (Plain Language)

Sanguine collects zero data. We operate zero servers. We have no ability to access, view, or retrieve your health information under any circumstances, including in response to legal process, because we do not possess it.

Your data is stored exclusively on your device in encrypted form. Backups, if enabled, are handled by your device's platform (Apple iCloud or Google Backup) and are encrypted by that platform — not by us. We have no access to those backups.

If you choose to export your data (as a backup file or a doctor report), that file is generated locally on your device and shared by you, to a destination you choose. Once shared, it is governed by the privacy practices of wherever you send it.

We do not use analytics, advertising SDKs, tracking pixels, device fingerprinting, or any mechanism that transmits information from your device to us or any third party.

This is not a policy. It is a description of how the software is built. The source code is open for verification.

---

## Status

Sanguine is in early design. This repository contains:

- **Design prototypes** exploring check-in UX, insight visualization, and severity tracking
- **This document** establishing the product values, architecture, and privacy commitments that will govern development

Development has not started.

---

## Name

*Sanguine* — from Latin *sanguis* (blood). In modern usage: optimistic, positive. A word that holds both the reality of what we track and the reason we track it.

---

## License

Sanguine is **source-available, not open source.** The code is public so anyone can verify our privacy claims. It is not licensed for use, modification, or redistribution. See [LICENSE.md](LICENSE.md) for full terms.
