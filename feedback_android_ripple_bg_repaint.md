---
name: android-ripple-bg-repaint
description: "Android won't reliably repaint a toggled backgroundColor in place — reuse SegmentedControl, or use an inner View with an inline bg + key it on active state to force remount."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4d2f856f-b4c5-4d20-95b6-66db38e9c3a9
---

In the Kowloon mobile app (React Native + NativeWind), a `Pressable` that has
`android_ripple` AND a dynamic `backgroundColor` (for a selected/active state)
will NOT repaint the background when the value changes on Android. The ripple
drawable IS that node's background layer, so the `backgroundColor` on the same
node gets masked and sticks on the previous value. This is true whether the bg
comes from a NativeWind `className` or an inline `style` — the node, not the
styling method, is the problem.

Symptom: tapping a new option in a segmented control / chip row leaves the old
highlight in place (two look selected), while the `<Text>` label color DOES
update (because it lives on the child Text, a different node).

**Fix:** move the dynamic `backgroundColor` onto a plain inner `<View>` (its own
layer, no ripple); keep `android_ripple` + `onPress` on the Pressable.

```jsx
<Pressable onPress={...} android_ripple={{ color: "rgba(0,0,0,0.08)" }}>
  <View style={{ backgroundColor: active ? "#5588B1" : "#FFFFFF" }}>
    <Text style={{ color: active ? "#F4F5F7" : "#1A1A20" }}>{label}</Text>
  </View>
</Pressable>
```

Applied 2026-07-18 to SegmentedControl, the typeface picker, CircleForm
visibility, bookmark FolderChip, and the notifications filter. An earlier fix
that only switched className→inline style on the Pressable did NOT work — Josh's
Typography screencast still showed the stuck highlight.

**ESCALATION (2026-07-27, preferences-screen pills #78):** even the documented
inner-`View` + inline-`backgroundColor` fix STILL wouldn't repaint in place on
Josh's device — only the `<Text>` label recolored. It cost several frustrating
round-trips. So harden the rule into a checklist for ANY toggling UI (chips,
pills, segmented controls, filter buttons):

1. **Prefer reusing `src/components/ui/SegmentedControl.jsx`** for single-select
   rows instead of hand-rolling a control.
2. If the selected state is expressible as **icon/text color only**, do that —
   `<Text>`/icon `color` on their own node always repaint (see
   `src/components/posts/TypeFilter.jsx`, which toggles `PostTypeIcon` color).
3. If you truly need a toggled **background**, put it on an inner `<View>` with an
   inline `style` AND **key that View on the active state** so it remounts on
   toggle: `<View key={active ? "on" : "off"} style={{ backgroundColor: ... }}>`.
   The remount is what guarantees the repaint when in-place restyle fails.

Never ship a background-toggle without testing it on-device, or (in-session)
without asking Josh to confirm it flips both ways. Related: [[design_aesthetic]],
[[project_preferences_screen]].
