# React Native TextInput — Complete Props Reference

A comprehensive reference for all `TextInput` properties in React Native.

---

## Table of Contents

1. [Core Props](#core-props)
2. [Keyboard & Input](#keyboard--input)
3. [Styling](#styling)
4. [Event Callbacks](#event-callbacks)
5. [iOS-Only Props](#ios-only-props)
6. [Android-Only Props](#android-only-props)

---

## Core Props

<details>
<summary><code>value</code> — <em>string</em></summary>

The current value of the input. Use with `onChangeText` to make the component controlled.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |

```jsx
<TextInput value={text} onChangeText={setText} />
```

</details>

<details>
<summary><code>defaultValue</code> — <em>string</em></summary>

Initial value for uncontrolled inputs. Not updated on re-render.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |

```jsx
<TextInput defaultValue="Hello" />
```

</details>

<details>
<summary><code>placeholder</code> — <em>string</em></summary>

Hint text shown when the input is empty.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |

```jsx
<TextInput placeholder="Enter your name" />
```

</details>

<details>
<summary><code>placeholderTextColor</code> — <em>color</em></summary>

Color of the placeholder text.

| | |
|---|---|
| **Type** | `color` |
| **Default** | system default |

```jsx
<TextInput placeholderTextColor="#aaa" />
```

</details>

<details>
<summary><code>editable</code> — <em>boolean</em></summary>

If `false`, the user cannot edit the text.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<TextInput editable={false} />
```

</details>

<details>
<summary><code>readOnly</code> — <em>boolean</em></summary>

Makes the field non-editable. Preferred over `editable={false}` in React 18+.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput readOnly />
```

</details>

<details>
<summary><code>maxLength</code> — <em>number</em></summary>

Limits the maximum number of characters that can be entered.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `undefined` |

```jsx
<TextInput maxLength={100} />
```

</details>

<details>
<summary><code>multiline</code> — <em>boolean</em></summary>

Allows the input to span multiple lines.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput multiline />
```

</details>

<details>
<summary><code>numberOfLines</code> — <em>number</em> · 🤖 Android</summary>

Sets the number of visible lines for a multiline `TextInput`.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `1` |
| **Platform** | Android |

```jsx
<TextInput multiline numberOfLines={4} />
```

</details>

<details>
<summary><code>secureTextEntry</code> — <em>boolean</em></summary>

Hides input text — useful for passwords. Disables `multiline`.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput secureTextEntry />
```

</details>

<details>
<summary><code>selectTextOnFocus</code> — <em>boolean</em></summary>

Automatically selects all text when the field receives focus.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput selectTextOnFocus />
```

</details>

<details>
<summary><code>selection</code> — <em>{ start: number, end?: number }</em></summary>

Controls the selection range. Use with `onSelectionChange`.

| | |
|---|---|
| **Type** | `{ start: number, end?: number }` |
| **Default** | `undefined` |

```jsx
<TextInput selection={{ start: 0, end: 4 }} />
```

</details>

<details>
<summary><code>selectionColor</code> — <em>color</em></summary>

Color of the selection highlight and cursor.

| | |
|---|---|
| **Type** | `color` |
| **Default** | system |

```jsx
<TextInput selectionColor="#6366f1" />
```

</details>

<details>
<summary><code>cursorColor</code> — <em>color</em> · 🤖 Android</summary>

Color of the cursor (blinking caret). iOS uses `selectionColor` for this purpose.

| | |
|---|---|
| **Type** | `color` |
| **Default** | system |
| **Platform** | Android |

```jsx
<TextInput cursorColor="#6366f1" />
```

</details>

<details>
<summary><code>caretHidden</code> — <em>boolean</em></summary>

If `true`, hides the cursor/caret.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput caretHidden />
```

</details>

<details>
<summary><code>autoFocus</code> — <em>boolean</em></summary>

Focuses the input automatically on mount.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<TextInput autoFocus />
```

</details>

<details>
<summary><code>allowFontScaling</code> — <em>boolean</em></summary>

Whether font scale from accessibility settings applies to this input.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<TextInput allowFontScaling={false} />
```

</details>

<details>
<summary><code>maxFontSizeMultiplier</code> — <em>number</em></summary>

Caps the multiplier for font scaling regardless of system settings.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `undefined` |

```jsx
<TextInput maxFontSizeMultiplier={1.5} />
```

</details>

<details>
<summary><code>ref</code> — <em>ref</em></summary>

Ref to access the underlying native input and call `.focus()`, `.blur()`, `.clear()`.

| | |
|---|---|
| **Type** | `ref` |
| **Default** | — |

```jsx
const ref = useRef();
<TextInput ref={ref} />
ref.current.focus();
```

</details>

<details>
<summary><code>testID</code> — <em>string</em></summary>

Used to locate this view in end-to-end tests.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |

```jsx
<TextInput testID="login-password" />
```

</details>

<details>
<summary><code>nativeID</code> — <em>string</em></summary>

Used to reference the component from native code.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |

```jsx
<TextInput nativeID="myInput" />
```

</details>

---

## Keyboard & Input

<details>
<summary><code>keyboardType</code> — <em>enum</em></summary>

Controls the type of keyboard shown.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'default'` |

**Values:** `default`, `number-pad`, `decimal-pad`, `numeric`, `email-address`, `phone-pad`, `url`, `ascii-capable` *(iOS)*, `numbers-and-punctuation` *(iOS)*, `name-phone-pad` *(iOS)*, `twitter` *(iOS)*, `web-search` *(iOS)*, `visible-password` *(Android)*

```jsx
<TextInput keyboardType="email-address" />
```

</details>

<details>
<summary><code>inputMode</code> — <em>enum</em></summary>

Modern cross-platform replacement for `keyboardType`.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'text'` |

**Values:** `none`, `text`, `decimal`, `numeric`, `tel`, `search`, `email`, `url`

```jsx
<TextInput inputMode="numeric" />
```

</details>

<details>
<summary><code>returnKeyType</code> — <em>enum</em></summary>

Label displayed on the return key.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'default'` |

**Values:** `done`, `go`, `next`, `search`, `send`, `none` *(Android)*, `previous` *(Android)*, `default`, `emergency-call` *(iOS)*, `google` *(Android)*, `join` *(iOS)*, `route` *(iOS)*, `yahoo` *(Android)*

```jsx
<TextInput returnKeyType="done" />
```

</details>

<details>
<summary><code>returnKeyLabel</code> — <em>string</em> · 🤖 Android</summary>

Sets a custom label on the return key.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |
| **Platform** | Android |

```jsx
<TextInput returnKeyLabel="Submit" />
```

</details>

<details>
<summary><code>keyboardAppearance</code> — <em>enum</em> · 🍎 iOS</summary>

Color scheme of the keyboard.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'default'` |
| **Platform** | iOS |

**Values:** `default`, `light`, `dark`

```jsx
<TextInput keyboardAppearance="dark" />
```

</details>

<details>
<summary><code>autoCapitalize</code> — <em>enum</em></summary>

Controls automatic capitalisation.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'sentences'` |

**Values:** `none`, `sentences`, `words`, `characters`

```jsx
<TextInput autoCapitalize="words" />
```

</details>

<details>
<summary><code>autoCorrect</code> — <em>boolean</em></summary>

Enables or disables autocorrect.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<TextInput autoCorrect={false} />
```

</details>

<details>
<summary><code>autoComplete</code> — <em>enum</em></summary>

Hints the autofill system about what data to suggest.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'off'` |

**Common values:** `off`, `username`, `password`, `email`, `name`, `tel`, `street-address`, `postal-code`, `cc-number`, `birthdate-full`, `one-time-code`, `new-password`

```jsx
<TextInput autoComplete="email" />
```

</details>

<details>
<summary><code>blurOnSubmit</code> — <em>boolean</em></summary>

Whether the input loses focus when the return key is pressed.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` (single-line), `false` (multiline) |

```jsx
<TextInput blurOnSubmit={false} />
```

</details>

<details>
<summary><code>submitBehavior</code> — <em>enum</em></summary>

Controls what happens when the return key is pressed.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'auto'` |

**Values:** `submit`, `newline`, `blurAndSubmit`

```jsx
<TextInput submitBehavior="submit" />
```

</details>

<details>
<summary><code>enablesReturnKeyAutomatically</code> — <em>boolean</em> · 🍎 iOS</summary>

Disables the return key when the input is empty; re-enables on text entry.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | iOS |

```jsx
<TextInput enablesReturnKeyAutomatically />
```

</details>

---

## Styling

<details>
<summary><code>style</code> — <em>StyleProp&lt;TextStyle&gt;</em></summary>

Style for the input. Accepts any `Text` and `View` style properties.

| | |
|---|---|
| **Type** | `StyleProp<TextStyle>` |
| **Default** | `{}` |

```jsx
<TextInput style={{ fontSize: 16, borderWidth: 1, padding: 8, borderRadius: 6 }} />
```

</details>

<details>
<summary><code>textAlign</code> — <em>enum</em></summary>

Aligns text within the input.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'left'` |

**Values:** `left`, `right`, `center`

```jsx
<TextInput textAlign="center" />
```

</details>

<details>
<summary><code>textAlignVertical</code> — <em>enum</em> · 🤖 Android</summary>

Vertical alignment for multiline inputs.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'auto'` |
| **Platform** | Android |

**Values:** `auto`, `top`, `bottom`, `center`

```jsx
<TextInput multiline textAlignVertical="top" />
```

</details>

<details>
<summary><code>textContentType</code> — <em>enum</em> · 🍎 iOS</summary>

Hints the system about semantic content for autofill.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'none'` |
| **Platform** | iOS |

**Common values:** `username`, `password`, `emailAddress`, `telephoneNumber`, `oneTimeCode`, `newPassword`, `name`, `fullStreetAddress`

```jsx
<TextInput textContentType="oneTimeCode" />
```

</details>

<details>
<summary><code>underlineColorAndroid</code> — <em>color</em> · 🤖 Android</summary>

Sets the color of the bottom border underline. Set `transparent` to remove it.

| | |
|---|---|
| **Type** | `color` |
| **Default** | system |
| **Platform** | Android |

```jsx
<TextInput underlineColorAndroid="transparent" />
```

</details>

<details>
<summary><code>inputAccessoryViewID</code> — <em>string</em> · 🍎 iOS</summary>

ID used to link an `InputAccessoryView` (toolbar above keyboard) to this input.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |
| **Platform** | iOS |

```jsx
<TextInput inputAccessoryViewID="doneBar" />
```

</details>

<details>
<summary><code>passwordRules</code> — <em>string</em> · 🍎 iOS</summary>

Describes password requirements for system password generation suggestions.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |
| **Platform** | iOS |

```jsx
<TextInput passwordRules="minlength: 8; required: digit;" />
```

</details>

<details>
<summary><code>smartInsertDelete</code> — <em>boolean</em> · 🍎 iOS</summary>

Controls auto-insert/delete of spaces around text pastes.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | iOS |

```jsx
<TextInput smartInsertDelete={false} />
```

</details>

<details>
<summary><code>lineBreakStrategyIOS</code> — <em>enum</em> · 🍎 iOS</summary>

Sets the line break strategy.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'none'` |
| **Platform** | iOS |

**Values:** `none`, `standard`, `hangul-word`, `push-out`

```jsx
<TextInput lineBreakStrategyIOS="standard" />
```

</details>

---

## Event Callbacks

<details>
<summary><code>onChangeText</code> — <em>(text: string) =&gt; void</em></summary>

Called on every text change with the new string value. The most commonly used callback.

```jsx
<TextInput onChangeText={(text) => setValue(text)} />
```

</details>

<details>
<summary><code>onChange</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called on change with the full native event object. Access text via `e.nativeEvent.text`.

```jsx
<TextInput onChange={(e) => console.log(e.nativeEvent.text)} />
```

</details>

<details>
<summary><code>onSubmitEditing</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the return/submit key is pressed.

```jsx
<TextInput onSubmitEditing={handleSubmit} />
```

</details>

<details>
<summary><code>onFocus</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the input receives focus.

```jsx
<TextInput onFocus={() => setFocused(true)} />
```

</details>

<details>
<summary><code>onBlur</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the input loses focus.

```jsx
<TextInput onBlur={() => setFocused(false)} />
```

</details>

<details>
<summary><code>onSelectionChange</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the selection changes. Receives `nativeEvent.selection` with `{ start, end }`.

```jsx
<TextInput onSelectionChange={(e) => setSelection(e.nativeEvent.selection)} />
```

</details>

<details>
<summary><code>onContentSizeChange</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the content size changes — useful for auto-growing multiline inputs.

```jsx
<TextInput
  multiline
  onContentSizeChange={(e) => setHeight(e.nativeEvent.contentSize.height)}
/>
```

</details>

<details>
<summary><code>onKeyPress</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called before each key press. `nativeEvent.key` contains the key name (e.g. `"a"`, `"Enter"`, `"Backspace"`).

```jsx
<TextInput onKeyPress={({ nativeEvent }) => console.log(nativeEvent.key)} />
```

</details>

<details>
<summary><code>onEndEditing</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when text input ends (user exits the field).

```jsx
<TextInput onEndEditing={handleEndEdit} />
```

</details>

<details>
<summary><code>onPressIn</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the user taps into the field.

```jsx
<TextInput onPressIn={() => setActive(true)} />
```

</details>

<details>
<summary><code>onPressOut</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when the user lifts their finger after tapping.

```jsx
<TextInput onPressOut={() => setActive(false)} />
```

</details>

<details>
<summary><code>onScroll</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called when content is scrolled inside a multiline input.

```jsx
<TextInput multiline onScroll={handleScroll} />
```

</details>

---

## iOS-Only Props

<details>
<summary><code>clearButtonMode</code> — <em>enum</em> · 🍎 iOS</summary>

Shows the clear (✕) button inside the field.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'never'` |
| **Platform** | iOS |

**Values:** `never`, `while-editing`, `unless-editing`, `always`

```jsx
<TextInput clearButtonMode="while-editing" />
```

</details>

<details>
<summary><code>clearTextOnFocus</code> — <em>boolean</em> · 🍎 iOS</summary>

Clears the text automatically when the field is focused.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | iOS |

```jsx
<TextInput clearTextOnFocus />
```

</details>

<details>
<summary><code>dataDetectorTypes</code> — <em>enum | enum[]</em> · 🍎 iOS</summary>

Detects data types and converts them to tappable links in multiline inputs.

| | |
|---|---|
| **Type** | `enum \| enum[]` |
| **Default** | `'none'` |
| **Platform** | iOS |

**Values:** `phoneNumber`, `link`, `address`, `calendarEvent`, `none`, `all`

```jsx
<TextInput multiline dataDetectorTypes={['phoneNumber', 'link']} />
```

</details>

<details>
<summary><code>scrollEnabled</code> — <em>boolean</em> · 🍎 iOS</summary>

Enables or disables scrolling in multiline inputs.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | iOS |

```jsx
<TextInput multiline scrollEnabled={false} />
```

</details>

<details>
<summary><code>spellCheck</code> — <em>boolean</em> · 🍎 iOS</summary>

Enables or disables spell checking independently of `autoCorrect`.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | inherited from `autoCorrect` |
| **Platform** | iOS |

```jsx
<TextInput spellCheck={false} />
```

</details>

<details>
<summary><code>rejectResponderTermination</code> — <em>boolean</em> · 🍎 iOS</summary>

When `true`, prevents other components from stealing the responder while the user is typing.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | iOS |

```jsx
<TextInput rejectResponderTermination />
```

</details>

<details>
<summary><code>contextMenuHidden</code> — <em>boolean</em> · 🍎 iOS</summary>

Hides the system context menu (cut/copy/paste).

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | iOS |

```jsx
<TextInput contextMenuHidden />
```

</details>

---

## Android-Only Props

<details>
<summary><code>disableFullscreenUI</code> — <em>boolean</em> · 🤖 Android</summary>

Prevents the OS from switching to fullscreen input mode on small screens.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | Android |

```jsx
<TextInput disableFullscreenUI />
```

</details>

<details>
<summary><code>importantForAutofill</code> — <em>enum</em> · 🤖 Android</summary>

Controls autofill visibility.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'auto'` |
| **Platform** | Android |

**Values:** `auto`, `no`, `noExcludeDescendants`, `yes`, `yesExcludeDescendants`

```jsx
<TextInput importantForAutofill="yes" />
```

</details>

<details>
<summary><code>inlineImageLeft</code> — <em>string</em> · 🤖 Android</summary>

Drawable resource name to show inline on the left side of the field.

| | |
|---|---|
| **Type** | `string` |
| **Default** | `undefined` |
| **Platform** | Android |

```jsx
<TextInput inlineImageLeft="search_icon" />
```

</details>

<details>
<summary><code>inlineImagePadding</code> — <em>number</em> · 🤖 Android</summary>

Padding between the inline image and the text.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0` |
| **Platform** | Android |

```jsx
<TextInput inlineImageLeft="search_icon" inlineImagePadding={8} />
```

</details>

<details>
<summary><code>rows</code> — <em>number</em> · 🤖 Android</summary>

Number of rows to display. Alias for `numberOfLines`.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `1` |
| **Platform** | Android |

```jsx
<TextInput rows={3} />
```

</details>

<details>
<summary><code>showSoftInputOnFocus</code> — <em>boolean</em> · 🤖 Android</summary>

If `false`, the soft keyboard does not appear on focus — useful for custom keyboard implementations.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | Android |

```jsx
<TextInput showSoftInputOnFocus={false} />
```

</details>

---

## Common Patterns

<details>
<summary>Auto-growing multiline input</summary>

```jsx
const [height, setHeight] = useState(40);

<TextInput
  multiline
  style={{ height, minHeight: 40, maxHeight: 200 }}
  onContentSizeChange={(e) =>
    setHeight(e.nativeEvent.contentSize.height)
  }
/>
```

</details>

<details>
<summary>OTP / SMS autofill (iOS)</summary>

```jsx
<TextInput
  keyboardType="number-pad"
  textContentType="oneTimeCode"
  maxLength={6}
  onChangeText={setOtp}
/>
```

</details>

<details>
<summary>Chained form fields with next focus</summary>

```jsx
<TextInput
  returnKeyType="next"
  blurOnSubmit={false}
  onSubmitEditing={() => passwordRef.current.focus()}
/>
<TextInput
  ref={passwordRef}
  secureTextEntry
  returnKeyType="done"
/>
```

</details>

<details>
<summary>Programmatic control via ref</summary>

```jsx
const inputRef = useRef(null);

<TextInput ref={inputRef} />

// Anywhere in your code:
inputRef.current.focus();
inputRef.current.blur();
inputRef.current.clear();
```

</details>

---

*Reference based on React Native stable API. Always check the [official docs](https://reactnative.dev/docs/textinput) for the latest updates.*