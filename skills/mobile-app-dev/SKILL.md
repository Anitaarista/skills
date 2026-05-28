---
name: mobile-app-dev
description: "Mobile application development for iOS and Android. Build cross-platform apps with React Native (Expo) or Flutter, or native apps with Kotlin (Android) and Swift (iOS). Use this skill whenever the user wants to create a mobile app, phone app, tablet app, React Native app, Flutter app, or any application targeting mobile devices. Also use this when the user mentions APK, IPA, App Store, Google Play, push notifications, camera access, GPS/location, or any mobile-specific feature. If the project is a responsive website (not a native mobile app), use flex-web-dev or fullstack-dev instead."
argument-hint: "Describe the mobile app you want to build, and optionally specify the platform/framework"
version: 1.0.0
---

# Mobile App Development Skill

Build mobile applications for iOS and Android. This skill supports multiple mobile development frameworks and guides the user through creating production-ready mobile apps.

## Core Philosophy

Mobile development requires different thinking than web development. Screen sizes are fixed, touch interactions replace mouse, performance matters more, and native device APIs are essential. This skill provides the patterns and best practices for building great mobile experiences.

## Environment Limitations

**IMPORTANT**: You are running in a sandbox environment. You can:
- Write and organize all source code for the mobile app
- Create project configuration files
- Set up the complete project structure
- Install npm dependencies
- Test the build process (if compatible)
- Generate asset files

**You CANNOT** in this environment:
- Run iOS Simulator (requires macOS + Xcode)
- Run Android Emulator (requires Android SDK + KVM)
- Build APK/IPA files (requires native build tools)
- Test on a physical device
- Run the Expo Go app

**Therefore**: Focus on creating a complete, well-structured project that the user can open on their own machine and run. Provide clear setup instructions.

## Tech Stack Selection

| Framework | Best For | Language | Hot Reload | Ecosystem |
|---|---|---|---|---|
| **React Native + Expo** (recommended) | Cross-platform, rapid development | JavaScript/TypeScript | Yes (Expo Go) | Massive |
| Flutter | Beautiful UI, high performance | Dart | Yes | Growing |
| React Native CLI | When you need custom native code | JavaScript/TypeScript | Yes | Massive |
| Kotlin (Android only) | Native Android apps | Kotlin | Via Compose | Native |
| Swift (iOS only) | Native iOS apps | Swift | Via SwiftUI | Native |

When the user doesn't specify, **recommend React Native + Expo** — it has the lowest barrier to entry, fastest development cycle, and largest community.

## Framework-Specific Guides

Read only the reference file for the chosen framework:
- `references/react-native-expo.md` — React Native with Expo (recommended default)
- `references/flutter.md` — Flutter with Dart
- `references/kotlin-android.md` — Native Android with Kotlin
- `references/swift-ios.md` — Native iOS with Swift

## Common Mobile Patterns (All Frameworks)

### Screen Architecture

Every mobile app needs a consistent screen architecture:
1. **Navigation** — Stack navigation, tab navigation, drawer
2. **State management** — Screen-level state + shared app state
3. **API integration** — Data fetching, caching, error handling
4. **Local storage** — SharedPreferences, AsyncStorage, SQLite
5. **Authentication** — Login, signup, token refresh, biometric auth

### UI/UX Guidelines for Mobile

- **Touch targets**: Minimum 44x44 points (iOS) / 48x48 dp (Android)
- **Font sizes**: Body text 16sp, headings 20-24sp, titles 28-34sp
- **Spacing**: 16dp padding standard, 8dp between related elements
- **Safe areas**: Respect notch/status bar/home indicator
- **Gestures**: Support swipe, pull-to-refresh, long-press where appropriate
- **Haptic feedback**: Use for important actions (delete, confirm, etc.)
- **Loading states**: Skeleton screens or shimmer effects over spinners
- **Empty states**: Show helpful messages and actions when content is empty
- **Error states**: Clear error messages with retry options

### Performance Best Practices

- **List virtualization**: Always use FlatList (RN) or ListView.builder (Flutter) for long lists
- **Image optimization**: Resize images to display size, use WebP format
- **Lazy loading**: Load data as the user scrolls, not all at once
- **Memoization**: Prevent unnecessary re-renders with memo/useMemo
- **Bundle size**: Tree-shake imports, avoid importing entire libraries
- **Network**: Cache responses, implement offline support

### Common Mobile Features

When implementing these features, read the appropriate reference file for framework-specific patterns:
- Camera / Photo picker
- GPS / Location services
- Push notifications (Firebase Cloud Messaging)
- Biometric authentication (fingerprint/face)
- Deep linking
- Background tasks
- Local notifications
- SQLite / local database
- File upload/download
- Real-time messaging (WebSocket)

## Project Setup Workflow

### Step 1: Create Project Structure
Generate the complete project with all necessary files organized properly.

### Step 2: Install Dependencies
Run `npm install` or equivalent. Verify no errors.

### Step 3: Write Core Code
Implement the screens, navigation, state management, and API integration.

### Step 4: Create Assets
Generate icons, splash screens, placeholder images using available tools.

### Step 5: Documentation
Create a `SETUP.md` file with:
- Prerequisites (Node.js version, etc.)
- How to install dependencies
- How to run on iOS simulator
- How to run on Android emulator
- How to build for production
- Environment variables needed
- Any native configuration steps (e.g., Firebase setup)

### Step 6: Save to Persistent Storage
Copy the entire project to `/home/z/my-project/upload/` so the user can download it:
```bash
cp -r /home/z/my-project/your-app /home/z/my-project/upload/your-app
```

## Important Rules

1. **Explain the limitation** — Tell the user upfront that you can create the complete code but they need to run it on their own machine
2. **Provide setup instructions** — Always create a SETUP.md with clear steps
3. **Use TypeScript** — Unless the user explicitly requests JavaScript
4. **Save to upload/** — Copy the final project to `/home/z/my-project/upload/` for download
5. **Cross-platform first** — Unless user specifies iOS or Android only, design for both
6. **Test the build** — At minimum, verify `npm install` and `npx expo export` (or equivalent) succeeds
7. **Mobile patterns, not web patterns** — Don't use web UI paradigms (hover states, fixed-width layouts) on mobile
