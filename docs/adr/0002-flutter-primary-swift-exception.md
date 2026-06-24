# ADR 0002: Flutter primary, Swift native exception

- **Date:** 2026-06-24
- **Status:** accepted

## Context

Mobile is a standard surface for Resonance Team products. The team needs a
default mobile framework that covers iOS + Android with minimal duplication.

## Decision

**Flutter** (Dart) is the primary mobile framework. It delivers one codebase
for iOS and Android, with Material 3 UI components and reasonable performance
for AI-application UIs (chat, forms, dashboards).

**Swift** (SwiftUI) is reserved for iOS-native work that Flutter cannot
handle: deep camera access, AR, wallet integration, widgets, App Clips, or
performance-critical animations. Using Swift is an exception that must be
documented with a project-level ADR explaining why Flutter was insufficient.

## Consequences

- **Positive:** One mobile codebase for most products; OTA updates via Dart
  code push; faster mobile iteration.
- **Positive:** Mobile team can be primarily Dart without iOS/Android
  platform specialists for most work.
- **Negative:** Dart is a third language in the team's stack (TS + Python +
  Dart + occasional Swift). Cross-language type sharing between TS and
  Dart requires a generated client or manual sync.
- **Negative:** Flutter has a larger binary size than native and may lag
  behind platform API adoption. The Swift exception clause covers this.
