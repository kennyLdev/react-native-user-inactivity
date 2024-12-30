<h1 align="center">react-native-user-inactivity</h1>
<p>
  <a href="https://github.com/kennyLdev/react-native-user-inactivity/actions/workflows/npm-publish-github-packages.yml">
    <img alt="Build Status" src="https://github.com/kennyldev/react-native-user-inactivity/actions/workflows/npm-publish-github-packages.yml/badge.svg" target="_blank" />
  </a>
  <a href="https://github.com/kennyldev/react-native-user-inactivity#readme">
    <img alt="Documentation" src="https://img.shields.io/badge/documentation-yes-brightgreen.svg" target="_blank" />
  </a>
  <a href="https://github.com/kennyldev/react-native-user-inactivity/graphs/commit-activity">
    <img alt="Maintenance" src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" target="_blank" />
  </a>
  <a href="https://github.com/kennyldev/react-native-user-inactivity/blob/master/LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" target="_blank" />
  </a>
</p>

## We forked this because we used it in some of our customer projects, and it prevented our movement of RN versions due to dependency requirements. We plan to maintain this version as needed. If you run into any issues, please don't hesitate to open an issue here on GitHub.

> Functional React Native component that notifies when the user stops interacting with the mobile screen for a given amount of time.

As of version 1.1.0, `react-native-user-inactivity` resets the timer also when the keyboard appears or disappears.
If you want to avoid this behaviour, you can set the `skipKeyboard` property to `true`.

As of version 1.0.0, `react-native-user-inactivity` has been rebuilt as a functional component that uses the new React Hook API.
Thanks to [`usetimeout-react-hook`](https://github.com/kennyldev/usetimeout-react-hook), `react-native-user-inactivity` supports timers different
than the standard one (`setTimeout`). This has solved some of the most recurrent issues, such as [#12](https://github.com/jkomyno/react-native-user-inactivity/issues/12), [#16](https://github.com/jkomyno/react-native-user-inactivity/issues/16), [#17](https://github.com/jkomyno/react-native-user-inactivity/issues/17).

## Install

```sh
npm install @kennyldev/react-native-user-inactivity
```

If you are running a version of react < 17 you'll need to include the `--legacy-peer-deps` flag.

```sh
npm install @kennyldev/react-native-user-inactivity --legacy-peer-deps
```

## 🔑 Key features

- 🥇 supports generic timers (you're no longer constrained to `setTimeout`)
- ⚠️ optional reset capability of the timer
- ✨ super elastic behaviour thanks to the Hooks API
- 💪 written in TypeScript
- ✔️ the core logic of this component is delegated to [`usetimeout-react-hook`](https://github.com/kennyldev/usetimeout-react-hook), which has 100% code coverage

## ❔ How to use

This package primarily exposes a single functional component, [UserInactivity](src/index.tsx).
The signature of the `UserInactivity` React props is the following:

```typescript
interface UserInactivityProps<T = unknown> {
  /**
   * Number of milliseconds after which the view is considered inactive.
   * If it changed, the timer restarts and the view is considered active until
   * the new timer expires.
   * It defaults to 1000.
   */
  timeForInactivity?: number;

  /**
   * If it's explicitly set to `true` after the component has already been initialized,
   * the timer restarts and the view is considered active until the new timer expires.
   * It defaults to true.
   */
  isActive?: boolean;

  /**
   * Generic usetimeout-react-hook's TimeoutHandler implementation.
   * It defaults to the standard setTimeout/clearTimeout implementation.
   * See https://github.com/jkomyno/usetimeout-react-hook/#-how-to-use.
   */
  timeoutHandler?: TimeoutHandler<T>;

  /**
   * Children components to embed inside UserInactivity's View.
   * If any children component is pressed, `onAction` is called after
   * `timeForInactivity` milliseconds.
   */
  children: React.ReactNode;

  /**
   * If set to true, the timer is not reset when the keyboard appears
   * or disappears.
   */
  skipKeyboard?: boolean;

  /**
   * Optional custom style for UserInactivity's View.
   * It defaults to { flex: 1 }.
   */
  style?: StyleProp<ViewStyle>;

  /**
   * Callback triggered anytime UserInactivity's View isn't touched for more than
   * `timeForInactivity` seconds.
   * It's `active` argument is true if and only if the View wasn't touched for more
   * than `timeForInactivity` milliseconds.
   */
  onAction: (active: boolean) => void;
}
```

When a native timer is needed (in order to avoid issues such as #12, #16, #17) an implementation of
[usetimeout-react-hook's TimeoutHandler](https://github.com/kennyldev/usetimeout-react-hook/#-how-to-use) should be
passed to the `timeoutHandler` prop.
A default one (BackgroundTimer) is optionally provided: in order to use it you must:

- manually run: `npm i -S react-native-background-timer`
- manually link the native library: `react-native link react-native-background-timer`

In case of doubts, please refer to the official [`react-native-background-timer`](https://github.com/ocetnik/react-native-background-timer) repository (moved to a [`fork`](https://github.com/boterop/react-native-background-timer)).

The default `BackgroundTimer` can be used like this:

```tsx
import UserInactivity from 'react-native-user-inactivity';
import BackgroundTimer from 'react-native-user-inactivity/lib/BackgroundTimer';

export default () => {
  return (
    <UserInactivity
      timeForInactivity={2000}
      timeoutHandler={BackgroundTimer}
      onAction={isActive => { console.log(isActive); }}
      style={{ flex: 1, paddingTop: '10%' }}
    >
  );
}
```

**Warning**: it seems that `react-native-background-timer` doesn't work properly with Android 10+ ([#41](https://github.com/jkomyno/react-native-user-inactivity/issues/41)). I'm currently unable to reproduce the problem, but help from the open-source community on this matter is certainly appreciated.

## ✨ Typings

Since the component itself is written in TypeScript, your editor's intellisense system should automagically detect
the typings file (even if you're using plain JS!), thus providing a better developer experience.
In fact, autocomplete capabilities and warning should come for free as you're typing the props to pass to the `UserInactivity` component.

## 💪 Practical Example

```tsx
import React, { useState } from "react";
import { View, Text, TextInput, Button } from "react-native";
import UserInactivity from "react-native-user-inactivity";

export default () => {
  const [active, setActive] = useState(true);
  const [timer, setTimer] = useState(2000);

  return (
    <View style={{ flex: 1 }}>
      <UserInactivity
        isActive={active}
        timeForInactivity={timer}
        onAction={(isActive) => {
          setActive(isActive);
        }}
        style={{ flex: 1, paddingTop: "10%" }}
      >
        <Button id="btn-1" title="1 Press this to simulate activity" />
        <Button id="btn-2" title="2 Press this to simulate activity" />
        <Text id="text-1" style={{ textAlign: "center" }}>
          Type below to simulate activity
        </Text>
        <TextInput
          id="text-input-1"
          style={{ height: 40, borderColor: "gray", borderWidth: 1 }}
          onChange={() => {
            setActive(true);
          }}
          textContentType="creditCardNumber"
          value={timer.toString(10)}
          onChangeText={(text) => setTimer(Number.parseInt(text || 0, 10))}
        />
      </UserInactivity>
      <View style={{ flex: 3, backgroundColor: "#fcfcaa" }}>
        <Text style={{ textAlign: "center" }}>
          {active ? "ACTIVE" : "NOT ACTIVE"}
        </Text>
        <Button
          title="Manually set to Active"
          onPress={() => {
            setActive(true);
          }}
        />
      </View>
    </View>
  );
};
```

Also, please checkout the [example on Snack/Expo](https://snack.expo.io/B1sjE9uMH).

---

## 🚀 Build package

This package is built using **TypeScript**, so the source needs to be converted in JavaScript before being usable by the users.
This can be achieved using TypeScript directly:

```sh
npm run build
```

## Current Maintainer

**Kenneth Lindelof**

- Github: [@kennyldev](https://github.com/kennyldev)

## 👤 Original Author

**Alberto Schiabel**

- Github: [@jkomyno](https://github.com/jkomyno)

## 🤝 Contributing

Contributions, issues and feature requests are welcome!<br />Feel free to check [issues page](https://github.com/kennyLdev/react-native-user-inactivity/issues).
The code is short, throughly commented and well tested, so you should feel quite comfortable working on it.
If you have any doubt or suggestion, please open an issue.

## 📝 License

This project is [MIT](https://github.com/kennyldev/usetimeout-react-hook/blob/master/LICENSE) licensed.

## Related packages

- [`usetimeout-react-hook`](https://github.com/kennyldev/usetimeout-react-hook)
