# Mobile App Testing Reference

This reference covers 10 mobile-specific testing domains. Use it when the test target is an iOS or Android application.

---

## 1. UI and Interaction Testing (UI 交互测试)

### Key Test Areas

- **Layout correctness**: Elements render in the correct position, size, and alignment across different screen sizes
- **Text rendering**: Font size, truncation, line wrapping, right-to-left languages, dynamic type (accessibility font scaling)
- **Touch targets**: Minimum 44×44pt (iOS) / 48×48dp (Android) for all tappable elements
- **Keyboard interaction**: Keyboard appears for correct input fields, "Next"/"Done" buttons work, keyboard doesn't obscure input, scroll-to-input behavior
- **Orientation**: Portrait and landscape rendering; transition doesn't lose state
- **Dark mode / light mode**: All elements are visible and readable in both themes
- **Scroll behavior**: Long content scrolls smoothly, pull-to-refresh works, infinite scroll loads more items
- **Input focus**: Visual focus indicator is clear, tab order is logical

### Common Defects to Watch

- Overlapping elements on small screens (iPhone SE, Android 5" devices)
- Text truncation in localized strings (German, Russian often 30-40% longer)
- Keyboard covering input fields without auto-scroll
- Status bar content not visible on certain background colors

---

## 2. Device Compatibility Matrix (设备兼容性矩阵)

### Test Dimensions

| Dimension | Examples |
|---|---|
| Screen sizes | Small (5"), Medium (6.1"), Large (6.7"), Tablet (10"-12") |
| OS versions | iOS 16/17/18, Android 12/13/14/15 |
| Manufacturers | Samsung, Xiaomi, Huawei, Google Pixel, OPPO (for Android) |
| Hardware | With/without notch, with/without physical home button, foldable devices |
| Memory | Low-end (2-3GB RAM), Mid-range (4-6GB), High-end (8GB+) |

### Minimum Coverage Recommendation

- Top 3 screen sizes × latest 2 OS versions = 6 configurations as baseline
- Add 1-2 low-end devices for performance testing
- Add foldable device if the app supports split-screen

### Test Cases to Include

- App launch and core flow on each device configuration
- Layout and rendering verification on smallest and largest screens
- Performance (startup time, memory) on lowest-spec device
- Feature availability checks for OS version-specific APIs (e.g., widgets, dynamic island)

---

## 3. Network Condition Testing (网络条件测试)

### Network Scenarios

| Scenario | How to Simulate | Key Checks |
|---|---|---|
| Strong Wi-Fi | Default connection | Baseline performance |
| 4G/LTE | Network throttle tool | Acceptable load times |
| 3G / Slow network | Throttle to 750kbps | Loading indicators shown, no timeouts |
| Offline | Airplane mode | Offline mode or clear error message |
| Wi-Fi → cellular switch | Toggle Wi-Fi off during operation | No data loss, request resumes or retries |
| Intermittent connectivity | Packet loss simulation (10-50%) | Graceful retry, no duplicate submissions |
| High latency | 2000-5000ms delay | Timeout handling, loading states |

### Common Defects

- Spinner shown indefinitely on slow network (no timeout)
- Duplicate orders submitted when user retries after apparent failure
- Cached data not shown when going offline
- App crash when network drops during file upload
- SSL certificate errors not handled gracefully

---

## 4. Push Notification Testing (推送通知测试)

### Test Scenarios

- **Permission flow**: First-time permission prompt appears correctly; denied permission shows appropriate fallback
- **Notification delivery**: Notification arrives when app is in foreground, background, and killed state
- **Notification content**: Title, body, badge count, icon, and sound are correct
- **Deep link from notification**: Tapping notification opens the correct screen with correct data
- **Rich notifications**: Image/video attachments display correctly
- **Notification grouping**: Multiple notifications are grouped properly (iOS) or use channels (Android)
- **Silent / data notifications**: Background data refresh triggers correctly
- **Do Not Disturb**: Notification queued and delivered when DND ends
- **Scheduled notifications**: Local notifications fire at the correct time, respecting timezone

### Edge Cases

- Notification received while user is actively using the target screen
- Expired or invalid deep link in notification payload
- Notification with extremely long text (truncation behavior)
- Multiple rapid notifications (flooding)

---

## 5. Permission Management (权限管理测试)

### Core Permissions to Test

| Permission | Test Scenarios |
|---|---|
| Camera | Grant, deny, "Ask Every Time" (iOS); use after revoke in Settings |
| Location | "While Using" vs "Always" vs "Never"; accuracy (precise vs approximate on Android) |
| Photos / Storage | Full access vs limited access (iOS 14+); read vs write on Android |
| Microphone | Grant and deny; in-call indicator |
| Notifications | Allow, deny, change in Settings → verify app behavior |
| Contacts | Grant, deny; partial access (iOS 18+) |
| Bluetooth | Grant, deny; needed for peripherals |

### Key Test Cases

- First-time permission request shows correct purpose string / rationale
- App functions correctly (with degraded features) when permission is denied
- Permission revoked mid-use: app does not crash, shows re-request prompt
- "Don't ask again" (Android): app detects this and directs user to Settings
- Multiple permissions requested: order and grouping make sense

---

## 6. App Lifecycle Testing (应用生命周期测试)

### States to Test

| State Transition | Trigger | Verify |
|---|---|---|
| Launch (cold start) | Tap app icon after force-quit | Splash screen → correct initial screen; login state preserved |
| Launch (warm start) | Return from recent apps | Previous screen restored with correct data |
| Background → Foreground | Home button then return | Data refreshes, session valid, no re-login |
| Interruptions | Phone call, alarm, system alert | App pauses gracefully, resumes from same state |
| Memory pressure | Open many other apps | App reloads gracefully if killed by OS, restores state |
| Force quit | Swipe away from task switcher | Next launch is a clean cold start |
| Split screen / PiP | Multi-window on supported devices | Layout adapts, functionality intact |

### Common Defects

- Form data lost when returning from background
- Video/audio playback doesn't resume after interruption
- Login session expired during background time → no clear re-login prompt
- Timer/countdown continues incorrectly after backgrounding

---

## 7. Install / Upgrade / Uninstall (安装/升级/卸载测试)

### Install Tests

- Fresh install from app store (or test distribution)
- Install on device with low storage → clear error message
- First launch after install: onboarding flow, permission requests, initial data download

### Upgrade Tests

- Upgrade from previous version (v N-1 → v N): data preserved, migration runs correctly
- Upgrade from much older version (v N-3 → v N): no data corruption
- Upgrade with pending background data (unsynced offline data)
- Database schema migration: verify data integrity after migration
- Settings and preferences preserved after upgrade

### Uninstall Tests

- Uninstall removes local data (verify no data remnants)
- Reinstall after uninstall: treated as fresh install
- Uninstall with active subscriptions: subscription status on server unaffected
- Keychain / secure storage data behavior after uninstall (iOS may retain keychain items)

---

## 8. Deep Linking (深链接测试)

### Test Scenarios

| Scenario | Verify |
|---|---|
| Universal link / App Link opens app to correct screen | Content matches the linked resource |
| Deep link when app is not installed | Redirects to app store or web fallback |
| Deep link when user is not logged in | Redirects to login → then navigates to target screen |
| Deep link with invalid/expired ID | Shows appropriate error, not a crash or blank screen |
| Deep link from email, SMS, web browser, social media | All entry points handled consistently |
| Deferred deep link | After install, user is directed to the original deep link target |

### Common Defects

- Deep link only works from certain referrers (e.g., works from Safari but not Chrome)
- Parameters lost during the redirect chain
- Deep link opens wrong screen when app was already open to a different screen
- URL encoding issues with special characters in deep link parameters

---

## 9. Gesture Testing (手势测试)

### Gestures to Test

| Gesture | Common Usage | Test Points |
|---|---|---|
| Tap | Select, toggle | Single tap responsiveness, accidental double-tap handling |
| Long press | Context menu, reorder | Press duration threshold, visual feedback during press |
| Swipe | Delete, navigation, dismiss | Direction recognition accuracy, partial swipe cancel |
| Pinch to zoom | Image, map | Smooth zoom, min/max zoom limits, double-tap to zoom |
| Pull to refresh | List refresh | Animation plays, data actually refreshes, disabled when inappropriate |
| Drag and drop | Reorder, move items | Drop targets highlight, cancel by dropping outside |
| Multi-finger | Rotate, multi-select | Two-finger gestures don't conflict with system gestures |
| Edge swipe | Back navigation (iOS) | App doesn't intercept system back gesture incorrectly |

### Common Defects

- Swipe-to-delete doesn't work near screen edges
- Pinch-to-zoom conflicts with scroll gesture on scrollable content
- Long press triggers both context menu and drag at the same time
- Pull-to-refresh animation stuck when network request fails

---

## 10. Mobile Accessibility (移动端无障碍测试)

### Screen Reader Testing (VoiceOver / TalkBack)

- All interactive elements have meaningful accessibility labels
- Reading order is logical (top-to-bottom, left-to-right or appropriate for locale)
- Custom controls are properly announced (role, state, value)
- Images have alt text; decorative images are hidden from screen reader
- Modals and overlays trap focus correctly; dismissing returns focus to trigger

### Other Accessibility Tests

| Area | Test Points |
|---|---|
| Dynamic type / font scaling | App remains usable at largest system font size; no text truncation that hides meaning |
| Color contrast | Minimum 4.5:1 ratio for normal text, 3:1 for large text (WCAG AA) |
| Color independence | Information not conveyed by color alone (e.g., error states also use icons/text) |
| Motion sensitivity | Reduced Motion setting respected; no auto-playing animations that can't be stopped |
| Touch accommodation | Works with assistive touch, switch control |
| Haptic feedback | Meaningful haptics that supplement (not replace) visual feedback |

### Minimum Accessibility Test Suite

1. Complete the core user flow using only VoiceOver (iOS) or TalkBack (Android)
2. Set system font to maximum size and verify all screens remain usable
3. Enable "Reduce Motion" and verify no vestibular-triggering animations play
4. Verify all form errors are announced by screen reader
5. Check that no interactive element is smaller than 44×44pt / 48×48dp
