# React Native Reference

Use this reference for React Native and Expo apps.

## Stack

- React Native
- Expo
- Expo Router when applicable
- TypeScript
- Uniwind / Tailwind
- React Native and Expo primitives
- Shared app components from `src/shared/components`

## Routing

Use `src/app/` for Expo Router route-level files.

```txt
src/app/
  _layout.tsx
  index.tsx
  (tabs)/
  (auth)/
```

Route files should mainly compose features and screens. Keep data access, validation, and feature state inside the feature slice.

Use `src/features/<feature>/ui/screens/` for feature screen-level views in React Native / Expo projects.

## UI

Use React Native and Expo primitives.

Do not use web UI kits such as MUI, Ant Design Web, Chakra UI, or browser-only components.

Good:

```tsx
import { Text, View } from "react-native";

const LoginScreen = () => {
  return (
    <View className="flex-1 bg-white px-4">
      <Text className="text-lg font-semibold text-black">Login</Text>
      <LoginForm />
    </View>
  );
};

export default LoginScreen;
```

## Styling

Use **Uniwind / Tailwind classes** for styling.

Good:

```tsx
<View className="flex-1 bg-white px-4">
  <Text className="text-lg font-semibold text-black">Title</Text>
</View>
```

Avoid inline styles unless required for dynamic runtime values.

Acceptable:

```tsx
<View style={{ width: progressWidth }} />
```

Bad:

```tsx
<View style={{ flex: 1, backgroundColor: "white", paddingHorizontal: 16 }} />
```

Prefer extracting repeated class groups into reusable components rather than duplicating large class strings.

## Platform-Specific Code

Keep platform-specific code explicit.

Recommended naming:

```txt
button.web.tsx
button.native.tsx
use-keyboard-insets.native.ts
use-viewport-size.web.ts
```

Use platform-specific files when behavior or UI differs between React web and React Native.

Avoid hiding large platform differences inside one messy component.
