```jsx
<Stack.Screen 
  name="YourScreenName"

  options={{
    // --- Header Content & Text ---
    title: 'Screen Title', // Fallback string for header title and tab/drawer labels
    headerTitle: (props) => <CustomTitleView {...props} />, // String or custom React component for the title
    headerBackTitle: 'Back', // iOS only: Text displayed next to the back arrow
    headerBackTitleVisible: true, // Whether the back button text should be visible
    headerLeft: (props) => <CustomLeftButton {...props} />, // Custom React component for the left side of the header
    headerRight: (props) => <CustomRightButton {...props} />, // Custom React component for the right side of the header
    headerTitleAlign: 'center', // Aligns the header title text ('left' | 'center')

    // --- Header Styling & Visibility ---
    headerShown: true, // Whether to show or hide the header for this screen
    headerStyle: {
      backgroundColor: '#6200ee', // Background color of the header
    },
    headerTintColor: '#ffffff', // Color of the header title text and back button icon
    headerTitleStyle: {
      fontWeight: 'bold', // Typography styles for the header title
      fontSize: 18,
    },
    headerBackTitleStyle: {
      fontSize: 16, // Typography styles for the back button text
    },
    headerTransparent: false, // If true, header floats on top of the screen content with absolute positioning
    headerBackground: () => <CustomBackgroundComponent />, // Custom React element to render behind the header
    headerBlurEffect: 'light', // iOS only: Applies a native blur effect to a transparent header ('extraLight' | 'light' | 'dark' | 'regular' | 'prominent')
    headerShadowVisible: true, // Toggles the bottom border/shadow line beneath the header

    // --- Layout & Window Behavior ---
    presentation: 'card', // Presentation style ('card' | 'modal' | 'transparentModal' | 'containedModal' | 'containedTransparentModal' | 'fullScreenModal')
    orientation: 'default', // Restricts screen orientation ('default' | 'all' | 'portrait' | 'landscape' | 'landscape_left' | 'landscape_right')
    statusBarHidden: false, // Whether to completely hide the device status bar at the top
    statusBarStyle: 'auto', // Color scheme of status bar text/icons ('auto' | 'inverted' | 'light' | 'dark')
    statusBarAnimation: 'fade', // Status bar transition effect ('fade' | 'slide' | 'none')
    contentStyle: {
      backgroundColor: '#f5f5f5', // Styles applied directly to the view wrapper wrapping your screen component
    },

    // --- Animations & Touch Gestures ---
    animation: 'default', // Entry transition style ('default' | 'fade' | 'fade_from_bottom' | 'flip' | 'slide_from_bottom' | 'slide_from_right' | 'slide_from_left' | 'none')
    animationDuration: 350, // Duration of the transition animation in milliseconds
    animationTypeForReplace: 'push', // Animation behavior when replacing a screen in history ('push' | 'pop')
    gestureEnabled: true, // Toggles whether swipe gestures can dismiss the screen
    gestureDirection: 'horizontal', // Sets swipe axis direction (primarily used in JS-based stack: 'horizontal' | 'vertical')

    // --- Native Device Integrations ---
    autoHideHomeIndicator: false, // iOS only: Automatically hides the native home indicator bar at the bottom
    freezeOnBlur: true, // Prevents the screen from re-rendering or consuming memory when in the background
    headerLargeTitle: false, // iOS only: Enables native expanding/collapsing large title layout
    headerLargeTitleStyle: {
      fontSize: 34, // Styles applied explicitly to the iOS native large title
      color: '#000000',
    },
    navigationBarColor: '#000000', // Android only: Background color of Android's system navigation bar (bottom buttons)
    navigationBarHidden: false, // Android only: Whether to hide the bottom system navigation bar
  }} 
/>
```