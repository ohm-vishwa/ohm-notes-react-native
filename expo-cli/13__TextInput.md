```jsx
<TextInput

// --- Value & State ---

value={text} // Controlled input value

defaultValue="Hello" // Initial value (uncontrolled)

onChangeText={(text) => setText(text)} // Called when text changes

onChange={(event) => {}} // Native change event

// --- Placeholder ---

placeholder="Enter text" // Placeholder text

placeholderTextColor="#999" // Placeholder color

// --- Keyboard ---

keyboardType="default" // Keyboard type

// "default"
// "numeric"
// "number-pad"
// "decimal-pad"
// "email-address"
// "phone-pad"
// "url"

returnKeyType="done" // Return key appearance

// "done"
// "go"
// "next"
// "search"
// "send"

keyboardAppearance="default" // iOS keyboard theme

// "default"
// "light"
// "dark"

showSoftInputOnFocus={true} // Show keyboard when focused


// --- Text Behavior ---

autoCapitalize="sentences" // Auto capitalization

// "none"
// "sentences"
// "words"
// "characters"

autoCorrect={true} // Enable autocorrect

spellCheck={true} // Enable spell checking

maxLength={100} // Maximum characters

editable={true} // Allow editing

readOnly={false} // Read-only mode

selectTextOnFocus={false} // Select all text on focus

clearTextOnFocus={false} // Clear text when focused

secureTextEntry={false} // Password field


// --- Focus & Blur ---

autoFocus={false} // Focus automatically

onFocus={() => {}} // Input focused

onBlur={() => {}} // Input lost focus

onEndEditing={() => {}} // Editing finished

onSubmitEditing={() => {}} // Return key pressed

blurOnSubmit={true} // Dismiss keyboard on submit


// --- Multiline ---

multiline={false} // Multiple lines

numberOfLines={4} // Visible lines (Android)

textAlignVertical="top" // Top align multiline text

onContentSizeChange={(e) => {}} // Content size changed


// --- Selection & Cursor ---

selectionColor="blue" // Highlight/cursor color

cursorColor="red" // Cursor color (Android)

selection={{
  start: 0,
  end: 5,
}} // Text selection range

onSelectionChange={(e) => {}} // Selection changed


// --- Styling ---

style={{
  borderWidth: 1,
  padding: 10,
}}

allowFontScaling={true} // Respect accessibility font size


// --- Events ---

onKeyPress={({ nativeEvent }) => {
  console.log(nativeEvent.key);
}} // Key pressed

onLayout={(event) => {}} // Layout changed


// --- Auto Complete ---

autoComplete="email" // Autofill hints

// "name"
// "email"
// "username"
// "password"
// "tel"
// "postal-code"
// etc.

textContentType="emailAddress" // iOS autofill type


// --- Android Specific ---

underlineColorAndroid="transparent" // Remove underline

importantForAutofill="yes" // Autofill importance


// --- Methods (Ref Required) ---

ref={inputRef} // Reference

// inputRef.current.focus()
// inputRef.current.blur()
// inputRef.current.clear()
// inputRef.current.isFocused()

/>
```