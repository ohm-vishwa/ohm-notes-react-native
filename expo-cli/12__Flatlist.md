# React Native FlatList — Complete Props Reference

A comprehensive reference for all `FlatList` properties in React Native.

---

## Table of Contents

1. [Core Props](#core-props)
2. [Layout & Columns](#layout--columns)
3. [Scroll Behavior](#scroll-behavior)
4. [Performance & Optimization](#performance--optimization)
5. [Headers, Footers & Separators](#headers-footers--separators)
6. [Empty & Loading States](#empty--loading-states)
7. [Pull to Refresh](#pull-to-refresh)
8. [Sticky Headers & Index](#sticky-headers--index)
9. [Event Callbacks](#event-callbacks)
10. [iOS-Only Props](#ios-only-props)
11. [Android-Only Props](#android-only-props)
12. [Common Patterns](#common-patterns)

---

## Core Props

<details>
<summary><code>data</code> — <em>ReadonlyArray&lt;ItemT&gt;</em> ⚠️ Required</summary>

The array of items to render. Each item is passed to `renderItem`.

| | |
|---|---|
| **Type** | `ReadonlyArray<ItemT>` |
| **Required** | Yes |

```jsx
<FlatList data={[{ id: '1', title: 'Item 1' }, { id: '2', title: 'Item 2' }]} />
```

</details>

<details>
<summary><code>renderItem</code> — <em>({ item, index, separators }) =&gt; ReactElement</em> ⚠️ Required</summary>

Function that renders each item. Receives `item`, `index`, and `separators` helper.

| | |
|---|---|
| **Type** | `({ item, index, separators }) => ReactElement` |
| **Required** | Yes |

```jsx
<FlatList
  data={data}
  renderItem={({ item }) => <Text>{item.title}</Text>}
/>
```

</details>

<details>
<summary><code>keyExtractor</code> — <em>(item: ItemT, index: number) =&gt; string</em></summary>

Extracts a unique key for each item. Used for caching and re-render optimization.

| | |
|---|---|
| **Type** | `(item, index) => string` |
| **Default** | Uses `item.key` or `item.id` fallback |

```jsx
<FlatList keyExtractor={(item) => item.id} />
```

</details>

<details>
<summary><code>extraData</code> — <em>any</em></summary>

A marker property to re-render the list when external state changes. Pass any state variable the `renderItem` depends on.

| | |
|---|---|
| **Type** | `any` |
| **Default** | `undefined` |

```jsx
const [selected, setSelected] = useState(null);
<FlatList extraData={selected} renderItem={...} />
```

</details>

<details>
<summary><code>horizontal</code> — <em>boolean</em></summary>

If `true`, renders items side by side in a horizontal scrolling list.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList horizontal data={data} renderItem={renderItem} />
```

</details>

<details>
<summary><code>inverted</code> — <em>boolean</em></summary>

Reverses the scroll direction. Useful for chat UIs where newest messages appear at the bottom.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList inverted data={messages} renderItem={renderMessage} />
```

</details>

<details>
<summary><code>initialNumToRender</code> — <em>number</em></summary>

Number of items to render in the initial batch. Enough to fill the screen without over-rendering.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `10` |

```jsx
<FlatList initialNumToRender={5} />
```

</details>

<details>
<summary><code>initialScrollIndex</code> — <em>number</em></summary>

Scrolls to this index on mount instead of starting at the top. Requires `getItemLayout`.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `undefined` |

```jsx
<FlatList initialScrollIndex={20} getItemLayout={getItemLayout} />
```

</details>

<details>
<summary><code>numColumns</code> — <em>number</em></summary>

Renders items in a grid with the given number of columns. Items must be of equal height.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `1` |

```jsx
<FlatList numColumns={3} data={photos} renderItem={renderPhoto} />
```

</details>

<details>
<summary><code>columnWrapperStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style applied to each row container when `numColumns > 1`.

| | |
|---|---|
| **Type** | `StyleProp<ViewStyle>` |
| **Default** | `undefined` |

```jsx
<FlatList
  numColumns={3}
  columnWrapperStyle={{ justifyContent: 'space-between' }}
/>
```

</details>

---

## Layout & Columns

<details>
<summary><code>getItemLayout</code> — <em>(data, index) =&gt; { length, offset, index }</em></summary>

Optimization that skips measurement of dynamic content. Use when all items have a fixed height (or width for horizontal).

| | |
|---|---|
| **Type** | `(data, index) => { length: number, offset: number, index: number }` |
| **Default** | `undefined` |

```jsx
<FlatList
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
/>
```

</details>

<details>
<summary><code>contentContainerStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style for the inner content container that wraps all items.

| | |
|---|---|
| **Type** | `StyleProp<ViewStyle>` |
| **Default** | `undefined` |

```jsx
<FlatList contentContainerStyle={{ padding: 16 }} />
```

</details>

<details>
<summary><code>style</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style for the outer scroll view container.

| | |
|---|---|
| **Type** | `StyleProp<ViewStyle>` |
| **Default** | `undefined` |

```jsx
<FlatList style={{ flex: 1, backgroundColor: '#fff' }} />
```

</details>

<details>
<summary><code>contentInset</code> — <em>{ top, bottom, left, right }</em> · 🍎 iOS</summary>

Insets the scroll content from the edges of the scroll view.

| | |
|---|---|
| **Type** | `{ top?: number, bottom?: number, left?: number, right?: number }` |
| **Default** | `{ top: 0, bottom: 0, left: 0, right: 0 }` |
| **Platform** | iOS |

```jsx
<FlatList contentInset={{ top: 64 }} />
```

</details>

<details>
<summary><code>contentOffset</code> — <em>{ x, y }</em></summary>

Sets the initial scroll offset.

| | |
|---|---|
| **Type** | `{ x: number, y: number }` |
| **Default** | `{ x: 0, y: 0 }` |

```jsx
<FlatList contentOffset={{ x: 0, y: 100 }} />
```

</details>

<details>
<summary><code>ItemSeparatorComponent</code> — <em>ComponentType</em></summary>

Rendered between each item (not before the first or after the last).

| | |
|---|---|
| **Type** | `ComponentType<{ highlighted, leadingItem }>` |
| **Default** | `undefined` |

```jsx
<FlatList
  ItemSeparatorComponent={() => <View style={{ height: 1, backgroundColor: '#eee' }} />}
/>
```

</details>

---

## Scroll Behavior

<details>
<summary><code>scrollEnabled</code> — <em>boolean</em></summary>

Enables or disables scrolling. Useful inside other scrollable containers.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<FlatList scrollEnabled={false} />
```

</details>

<details>
<summary><code>pagingEnabled</code> — <em>boolean</em></summary>

Snaps the scroll to multiples of the scroll view's size — useful for carousels.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList pagingEnabled horizontal />
```

</details>

<details>
<summary><code>snapToInterval</code> — <em>number</em></summary>

Snaps scrolling to multiples of this value. Alternative to `pagingEnabled` for custom item sizes.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `undefined` |

```jsx
<FlatList snapToInterval={320} decelerationRate="fast" />
```

</details>

<details>
<summary><code>snapToOffsets</code> — <em>number[]</em></summary>

Snaps to specific scroll offsets. Overrides `snapToInterval`.

| | |
|---|---|
| **Type** | `number[]` |
| **Default** | `undefined` |

```jsx
<FlatList snapToOffsets={[100, 300, 600]} />
```

</details>

<details>
<summary><code>snapToAlignment</code> — <em>enum</em> · 🍎 iOS</summary>

Defines alignment of snapping relative to the scroll view.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'start'` |
| **Platform** | iOS |

**Values:** `start`, `center`, `end`

```jsx
<FlatList snapToInterval={300} snapToAlignment="center" />
```

</details>

<details>
<summary><code>decelerationRate</code> — <em>number | enum</em></summary>

Controls how quickly the scroll slows after the user lifts their finger.

| | |
|---|---|
| **Type** | `'fast' \| 'normal' \| number` |
| **Default** | `'normal'` |

```jsx
<FlatList decelerationRate="fast" />
```

</details>

<details>
<summary><code>showsVerticalScrollIndicator</code> — <em>boolean</em></summary>

Shows or hides the vertical scroll bar.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<FlatList showsVerticalScrollIndicator={false} />
```

</details>

<details>
<summary><code>showsHorizontalScrollIndicator</code> — <em>boolean</em></summary>

Shows or hides the horizontal scroll bar.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |

```jsx
<FlatList showsHorizontalScrollIndicator={false} />
```

</details>

<details>
<summary><code>bounces</code> — <em>boolean</em> · 🍎 iOS</summary>

Enables the bounce effect when scrolling beyond content bounds.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | iOS |

```jsx
<FlatList bounces={false} />
```

</details>

<details>
<summary><code>overScrollMode</code> — <em>enum</em> · 🤖 Android</summary>

Controls over-scroll behavior on Android.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'auto'` |
| **Platform** | Android |

**Values:** `auto`, `always`, `never`

```jsx
<FlatList overScrollMode="never" />
```

</details>

<details>
<summary><code>scrollEventThrottle</code> — <em>number</em> · 🍎 iOS</summary>

Controls how often the scroll event fires (in ms). Lower = more frequent. Set to `16` for smooth tracking.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0` (fires only on each frame) |
| **Platform** | iOS |

```jsx
<FlatList scrollEventThrottle={16} onScroll={handleScroll} />
```

</details>

<details>
<summary><code>keyboardDismissMode</code> — <em>enum</em></summary>

Controls whether and how the keyboard dismisses on scroll.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'none'` |

**Values:** `none`, `on-drag`, `interactive` *(iOS only)*

```jsx
<FlatList keyboardDismissMode="on-drag" />
```

</details>

<details>
<summary><code>keyboardShouldPersistTaps</code> — <em>enum</em></summary>

Controls whether the keyboard stays up after a tap outside the input.

| | |
|---|---|
| **Type** | `enum` |
| **Default** | `'never'` |

**Values:** `never`, `always`, `handled`

```jsx
<FlatList keyboardShouldPersistTaps="handled" />
```

</details>

---

## Performance & Optimization

<details>
<summary><code>maxToRenderPerBatch</code> — <em>number</em></summary>

Maximum items rendered per batch during incremental rendering. Higher = smoother but heavier.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `10` |

```jsx
<FlatList maxToRenderPerBatch={5} />
```

</details>

<details>
<summary><code>updateCellsBatchingPeriod</code> — <em>number</em></summary>

Time in ms between batched renders. Lower = more responsive but more CPU load.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `50` |

```jsx
<FlatList updateCellsBatchingPeriod={30} />
```

</details>

<details>
<summary><code>windowSize</code> — <em>number</em></summary>

The render window size in units of visible length. Items outside are unmounted. `1` = visible area only; `21` = 10 screens above and below.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `21` |

```jsx
<FlatList windowSize={5} />
```

</details>

<details>
<summary><code>removeClippedSubviews</code> — <em>boolean</em></summary>

Detaches offscreen views from the native view hierarchy. Can improve performance on large lists.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList removeClippedSubviews />
```

</details>

<details>
<summary><code>disableVirtualization</code> — <em>boolean</em> ⚠️ Deprecated</summary>

Disables virtualization entirely. Deprecated — avoid using this.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Status** | Deprecated |

```jsx
// Avoid — use windowSize instead
<FlatList disableVirtualization />
```

</details>

<details>
<summary><code>legacyImplementation</code> — <em>boolean</em> ⚠️ Deprecated</summary>

Uses the older `ScrollView`-based implementation. Deprecated.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Status** | Deprecated |

```jsx
// Avoid in new code
<FlatList legacyImplementation />
```

</details>

---

## Headers, Footers & Separators

<details>
<summary><code>ListHeaderComponent</code> — <em>ComponentType | ReactElement</em></summary>

Rendered at the very top of the list, above all items.

| | |
|---|---|
| **Type** | `ComponentType \| ReactElement` |
| **Default** | `undefined` |

```jsx
<FlatList
  ListHeaderComponent={<Text style={{ fontSize: 20 }}>My List</Text>}
/>
```

</details>

<details>
<summary><code>ListHeaderComponentStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style applied to the wrapper of `ListHeaderComponent`.

| | |
|---|---|
| **Type** | `StyleProp<ViewStyle>` |
| **Default** | `undefined` |

```jsx
<FlatList ListHeaderComponentStyle={{ padding: 16, backgroundColor: '#f5f5f5' }} />
```

</details>

<details>
<summary><code>ListFooterComponent</code> — <em>ComponentType | ReactElement</em></summary>

Rendered at the very bottom of the list, below all items. Commonly used for load-more spinners.

| | |
|---|---|
| **Type** | `ComponentType \| ReactElement` |
| **Default** | `undefined` |

```jsx
<FlatList
  ListFooterComponent={isLoading ? <ActivityIndicator /> : null}
/>
```

</details>

<details>
<summary><code>ListFooterComponentStyle</code> — <em>StyleProp&lt;ViewStyle&gt;</em></summary>

Style applied to the wrapper of `ListFooterComponent`.

| | |
|---|---|
| **Type** | `StyleProp<ViewStyle>` |
| **Default** | `undefined` |

```jsx
<FlatList ListFooterComponentStyle={{ padding: 24 }} />
```

</details>

<details>
<summary><code>ItemSeparatorComponent</code> — <em>ComponentType</em></summary>

Rendered between each item. Receives `highlighted` and `leadingItem` props.

| | |
|---|---|
| **Type** | `ComponentType<{ highlighted: boolean, leadingItem: ItemT }>` |
| **Default** | `undefined` |

```jsx
<FlatList
  ItemSeparatorComponent={({ highlighted }) => (
    <View style={[styles.separator, highlighted && styles.highlighted]} />
  )}
/>
```

</details>

---

## Empty & Loading States

<details>
<summary><code>ListEmptyComponent</code> — <em>ComponentType | ReactElement</em></summary>

Rendered when `data` is empty. Great for empty state illustrations or messages.

| | |
|---|---|
| **Type** | `ComponentType \| ReactElement` |
| **Default** | `undefined` |

```jsx
<FlatList
  data={items}
  ListEmptyComponent={<Text>No items found.</Text>}
/>
```

</details>

---

## Pull to Refresh

<details>
<summary><code>refreshing</code> — <em>boolean</em></summary>

Controls the visibility of the pull-to-refresh spinner. Set to `true` while fetching.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList refreshing={isRefreshing} onRefresh={handleRefresh} />
```

</details>

<details>
<summary><code>onRefresh</code> — <em>() =&gt; void</em></summary>

Callback triggered when the user pulls down to refresh. Must be used with `refreshing`.

| | |
|---|---|
| **Type** | `() => void` |
| **Default** | `undefined` |

```jsx
<FlatList
  refreshing={isRefreshing}
  onRefresh={async () => {
    setIsRefreshing(true);
    await fetchData();
    setIsRefreshing(false);
  }}
/>
```

</details>

<details>
<summary><code>refreshControl</code> — <em>ReactElement</em></summary>

Custom `RefreshControl` component for full control over colors and behavior. Replaces `refreshing` + `onRefresh`.

| | |
|---|---|
| **Type** | `ReactElement` |
| **Default** | `undefined` |

```jsx
<FlatList
  refreshControl={
    <RefreshControl
      refreshing={refreshing}
      onRefresh={onRefresh}
      tintColor="#6366f1"
      colors={['#6366f1']}
    />
  }
/>
```

</details>

<details>
<summary><code>progressViewOffset</code> — <em>number</em> · 🤖 Android</summary>

Vertical offset for the pull-to-refresh spinner on Android.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0` |
| **Platform** | Android |

```jsx
<FlatList progressViewOffset={64} refreshing={refreshing} onRefresh={onRefresh} />
```

</details>

---

## Sticky Headers & Index

<details>
<summary><code>stickyHeaderIndices</code> — <em>number[]</em></summary>

Array of item indices that stick to the top of the scroll view when scrolling past them.

| | |
|---|---|
| **Type** | `number[]` |
| **Default** | `undefined` |

```jsx
<FlatList stickyHeaderIndices={[0]} />
```

</details>

<details>
<summary><code>stickyHeaderHiddenOnScroll</code> — <em>boolean</em></summary>

If `true`, the sticky header hides when scrolling down and reveals when scrolling up.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList stickyHeaderIndices={[0]} stickyHeaderHiddenOnScroll />
```

</details>

<details>
<summary><code>invertStickyHeaders</code> — <em>boolean</em></summary>

If `true`, sticky headers stick to the bottom instead of the top. Used with `inverted`.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |

```jsx
<FlatList inverted invertStickyHeaders stickyHeaderIndices={[0]} />
```

</details>

---

## Event Callbacks

<details>
<summary><code>onScroll</code> — <em>(event: NativeSyntheticEvent) =&gt; void</em></summary>

Called as the user scrolls. Use with `scrollEventThrottle` to control frequency.

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent<NativeScrollEvent>) => void` |

```jsx
<FlatList
  scrollEventThrottle={16}
  onScroll={({ nativeEvent }) => {
    console.log(nativeEvent.contentOffset.y);
  }}
/>
```

</details>

<details>
<summary><code>onScrollBeginDrag</code> — <em>(event) =&gt; void</em></summary>

Called when the user starts dragging the scroll view.

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent) => void` |

```jsx
<FlatList onScrollBeginDrag={() => dismissKeyboard()} />
```

</details>

<details>
<summary><code>onScrollEndDrag</code> — <em>(event) =&gt; void</em></summary>

Called when the user stops dragging and releases the scroll view.

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent) => void` |

```jsx
<FlatList onScrollEndDrag={handleScrollEnd} />
```

</details>

<details>
<summary><code>onMomentumScrollBegin</code> — <em>(event) =&gt; void</em></summary>

Called when momentum scrolling starts (after the user releases).

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent) => void` |

```jsx
<FlatList onMomentumScrollBegin={handleMomentumStart} />
```

</details>

<details>
<summary><code>onMomentumScrollEnd</code> — <em>(event) =&gt; void</em></summary>

Called when momentum scrolling ends.

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent) => void` |

```jsx
<FlatList onMomentumScrollEnd={handleMomentumEnd} />
```

</details>

<details>
<summary><code>onEndReached</code> — <em>({ distanceFromEnd }) =&gt; void</em></summary>

Called when scrolling reaches the end of the list. Use to trigger pagination / load more.

| | |
|---|---|
| **Type** | `({ distanceFromEnd: number }) => void` |
| **Default** | `undefined` |

```jsx
<FlatList
  onEndReached={({ distanceFromEnd }) => {
    if (distanceFromEnd < 0) return;
    loadMoreItems();
  }}
  onEndReachedThreshold={0.5}
/>
```

</details>

<details>
<summary><code>onEndReachedThreshold</code> — <em>number</em></summary>

How far from the end (in units of visible length) to trigger `onEndReached`. `0.5` = halfway to the end.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `2` |

```jsx
<FlatList onEndReachedThreshold={0.3} onEndReached={loadMore} />
```

</details>

<details>
<summary><code>onViewableItemsChanged</code> — <em>({ viewableItems, changed }) =&gt; void</em></summary>

Called when visibility of items changes. Use with `viewabilityConfig`.

| | |
|---|---|
| **Type** | `({ viewableItems: ViewToken[], changed: ViewToken[] }) => void` |
| **Default** | `undefined` |

```jsx
const onViewableItemsChanged = useRef(({ viewableItems }) => {
  console.log('Visible:', viewableItems.map(v => v.item.id));
});

<FlatList
  onViewableItemsChanged={onViewableItemsChanged.current}
  viewabilityConfig={{ itemVisiblePercentThreshold: 50 }}
/>
```

</details>

<details>
<summary><code>viewabilityConfig</code> — <em>ViewabilityConfig</em></summary>

Defines what counts as "viewable" for `onViewableItemsChanged`.

| | |
|---|---|
| **Type** | `ViewabilityConfig` |
| **Default** | `undefined` |

```jsx
<FlatList
  viewabilityConfig={{
    minimumViewTime: 300,
    itemVisiblePercentThreshold: 75,
    waitForInteraction: true,
  }}
/>
```

</details>

<details>
<summary><code>viewabilityConfigCallbackPairs</code> — <em>ViewabilityConfigCallbackPairs</em></summary>

Multiple viewability/callback pairs. Alternative to `viewabilityConfig` + `onViewableItemsChanged`.

| | |
|---|---|
| **Type** | `ViewabilityConfigCallbackPairs` |
| **Default** | `undefined` |

```jsx
<FlatList
  viewabilityConfigCallbackPairs={[
    { viewabilityConfig: { itemVisiblePercentThreshold: 50 }, onViewableItemsChanged: handler1 },
    { viewabilityConfig: { minimumViewTime: 1000 }, onViewableItemsChanged: handler2 },
  ]}
/>
```

</details>

<details>
<summary><code>onLayout</code> — <em>(event) =&gt; void</em></summary>

Called when the component layout changes.

| | |
|---|---|
| **Type** | `(event: LayoutChangeEvent) => void` |

```jsx
<FlatList onLayout={(e) => setListHeight(e.nativeEvent.layout.height)} />
```

</details>

<details>
<summary><code>onContentSizeChange</code> — <em>(width, height) =&gt; void</em></summary>

Called when the scrollable content size changes.

| | |
|---|---|
| **Type** | `(contentWidth: number, contentHeight: number) => void` |

```jsx
<FlatList onContentSizeChange={(w, h) => setContentHeight(h)} />
```

</details>

---

## iOS-Only Props

<details>
<summary><code>automaticallyAdjustContentInsets</code> — <em>boolean</em> · 🍎 iOS</summary>

Allows iOS to automatically adjust content insets for navigation bars and tab bars.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | iOS |

```jsx
<FlatList automaticallyAdjustContentInsets={false} />
```

</details>

<details>
<summary><code>automaticallyAdjustsScrollIndicatorInsets</code> — <em>boolean</em> · 🍎 iOS</summary>

Whether to adjust scroll indicator insets automatically.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` |
| **Platform** | iOS |

```jsx
<FlatList automaticallyAdjustsScrollIndicatorInsets={false} />
```

</details>

<details>
<summary><code>directionalLockEnabled</code> — <em>boolean</em> · 🍎 iOS</summary>

Locks scrolling to one axis once dragging starts.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` |
| **Platform** | iOS |

```jsx
<FlatList directionalLockEnabled />
```

</details>

<details>
<summary><code>alwaysBounceVertical</code> — <em>boolean</em> · 🍎 iOS</summary>

Always bounces vertically, even if the content is shorter than the view.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `false` (vertical list), `true` (horizontal) |
| **Platform** | iOS |

```jsx
<FlatList alwaysBounceVertical={false} />
```

</details>

<details>
<summary><code>alwaysBounceHorizontal</code> — <em>boolean</em> · 🍎 iOS</summary>

Always bounces horizontally, even if the content is shorter than the view.

| | |
|---|---|
| **Type** | `boolean` |
| **Default** | `true` (horizontal list), `false` (vertical) |
| **Platform** | iOS |

```jsx
<FlatList horizontal alwaysBounceHorizontal />
```

</details>

<details>
<summary><code>scrollIndicatorInsets</code> — <em>{ top, bottom, left, right }</em> · 🍎 iOS</summary>

Insets for the scroll indicator (the thin bar on the edge).

| | |
|---|---|
| **Type** | `{ top?: number, bottom?: number, left?: number, right?: number }` |
| **Default** | `{ 0, 0, 0, 0 }` |
| **Platform** | iOS |

```jsx
<FlatList scrollIndicatorInsets={{ right: 1 }} />
```

</details>

<details>
<summary><code>onScrollToTop</code> — <em>(event) =&gt; void</em> · 🍎 iOS</summary>

Called when the user taps the status bar to scroll to the top.

| | |
|---|---|
| **Type** | `(event: NativeSyntheticEvent) => void` |
| **Platform** | iOS |

```jsx
<FlatList onScrollToTop={() => console.log('scrolled to top')} />
```

</details>

---

## Android-Only Props

<details>
<summary><code>fadingEdgeLength</code> — <em>number</em> · 🤖 Android</summary>

Fades out the edges of the list to indicate more content. Set to `0` to disable.

| | |
|---|---|
| **Type** | `number` |
| **Default** | `0` |
| **Platform** | Android |

```jsx
<FlatList fadingEdgeLength={32} />
```

</details>

<details>
<summary><code>endFillColor</code> — <em>color</em> · 🤖 Android</summary>

Fills the space at the end of the scroll view when content is shorter than the view.

| | |
|---|---|
| **Type** | `color` |
| **Default** | `undefined` |
| **Platform** | Android |

```jsx
<FlatList endFillColor="#f9f9f9" />
```

</details>

---

## Imperative Methods (via ref)

<details>
<summary><code>scrollToIndex</code> — <em>method</em></summary>

Scrolls to the item at the given index. Requires `getItemLayout` for reliable behavior.

```jsx
const listRef = useRef(null);

listRef.current.scrollToIndex({ index: 10, animated: true });
// Options: { index, animated?, viewOffset?, viewPosition? }
```

</details>

<details>
<summary><code>scrollToItem</code> — <em>method</em></summary>

Scrolls to a specific item object. Less efficient than `scrollToIndex` — prefer index when possible.

```jsx
listRef.current.scrollToItem({ item: myItem, animated: true });
```

</details>

<details>
<summary><code>scrollToOffset</code> — <em>method</em></summary>

Scrolls to a specific pixel offset in the list.

```jsx
listRef.current.scrollToOffset({ offset: 500, animated: true });
```

</details>

<details>
<summary><code>scrollToEnd</code> — <em>method</em></summary>

Scrolls to the very end of the list.

```jsx
listRef.current.scrollToEnd({ animated: true });
```

</details>

<details>
<summary><code>recordInteraction</code> — <em>method</em></summary>

Tells the list an interaction has occurred, which may trigger deferred rendering.

```jsx
listRef.current.recordInteraction();
```

</details>

<details>
<summary><code>flashScrollIndicators</code> — <em>method</em></summary>

Briefly shows the scroll indicators.

```jsx
listRef.current.flashScrollIndicators();
```

</details>

---

## Common Patterns

<details>
<summary>Infinite scroll / load more on end</summary>

```jsx
const [data, setData] = useState([]);
const [loading, setLoading] = useState(false);
const [page, setPage] = useState(1);

const loadMore = async () => {
  if (loading) return;
  setLoading(true);
  const newItems = await fetchPage(page + 1);
  setData(prev => [...prev, ...newItems]);
  setPage(p => p + 1);
  setLoading(false);
};

<FlatList
  data={data}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  onEndReached={loadMore}
  onEndReachedThreshold={0.5}
  ListFooterComponent={loading ? <ActivityIndicator /> : null}
/>
```

</details>

<details>
<summary>Pull-to-refresh</summary>

```jsx
const [refreshing, setRefreshing] = useState(false);

const onRefresh = async () => {
  setRefreshing(true);
  await fetchData();
  setRefreshing(false);
};

<FlatList
  data={data}
  renderItem={renderItem}
  refreshing={refreshing}
  onRefresh={onRefresh}
/>
```

</details>

<details>
<summary>Photo grid (numColumns)</summary>

```jsx
const COLS = 3;
const SIZE = Dimensions.get('window').width / COLS;

<FlatList
  data={photos}
  keyExtractor={(item) => item.id}
  numColumns={COLS}
  columnWrapperStyle={{ gap: 2 }}
  renderItem={({ item }) => (
    <Image source={{ uri: item.uri }} style={{ width: SIZE, height: SIZE }} />
  )}
/>
```

</details>

<details>
<summary>Horizontal carousel with paging</summary>

```jsx
<FlatList
  data={slides}
  renderItem={({ item }) => <SlideCard item={item} />}
  horizontal
  pagingEnabled
  showsHorizontalScrollIndicator={false}
  keyExtractor={(item) => item.id}
/>
```

</details>

<details>
<summary>Chat UI (inverted list)</summary>

```jsx
<FlatList
  data={messages}
  renderItem={({ item }) => <MessageBubble message={item} />}
  keyExtractor={(item) => item.id}
  inverted
  keyboardDismissMode="interactive"
  keyboardShouldPersistTaps="handled"
/>
```

</details>

<details>
<summary>Track visible items (analytics / autoplay)</summary>

```jsx
const onViewableItemsChanged = useRef(({ viewableItems }) => {
  const visible = viewableItems.filter(v => v.isViewable);
  trackImpressions(visible.map(v => v.item.id));
}).current;

const viewabilityConfig = useRef({
  itemVisiblePercentThreshold: 50,
  minimumViewTime: 500,
}).current;

<FlatList
  data={feed}
  renderItem={renderItem}
  onViewableItemsChanged={onViewableItemsChanged}
  viewabilityConfig={viewabilityConfig}
/>
```

</details>

<details>
<summary>Scroll to top via ref</summary>

```jsx
const listRef = useRef(null);

const scrollToTop = () => {
  listRef.current?.scrollToOffset({ offset: 0, animated: true });
};

<FlatList ref={listRef} data={data} renderItem={renderItem} />
<Button title="Back to top" onPress={scrollToTop} />
```

</details>

<details>
<summary>Fixed-height items with getItemLayout (performance)</summary>

```jsx
const ITEM_HEIGHT = 72;

<FlatList
  data={data}
  renderItem={renderItem}
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  initialScrollIndex={50}
/>
```

</details>

---


> [!note]- React Native TextInput
> Hidden content here

*Reference based on React Native stable API. Always check the [official docs](https://reactnative.dev/docs/flatlist) for the latest updates.*