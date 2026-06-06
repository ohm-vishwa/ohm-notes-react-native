```jsx
<FlatList
  // --- Core Data & Rendering ---
  data={dataArray} // Array of items to render
  renderItem={({ item, index, separators }) => (
    <CustomItem item={item} index={index} />
  )} // Function that renders each item
  keyExtractor={(item, index) => item.id.toString()} // Unique key for each item
  // --- Layout & Orientation ---
  horizontal={false} // Render items horizontally instead of  vertically
  numColumns={1} // Number of columns for grid layout
  columnWrapperStyle={{
    justifyContent: "space-between",
  }} // Style for each row when using numColumns
  inverted={false} // Reverses scroll direction
  contentContainerStyle={{
    padding: 16,
  }} // Styling for inner content container
  style={{
    flex: 1,
  }} // Style for FlatList itself
  // --- Header & Footer ---
  ListHeaderComponent={<Header />} // Component rendered at top
  ListHeaderComponentStyle={{
    marginBottom: 10,
  }} // Style for header
  ListFooterComponent={<Footer />} // Component rendered at bottom
  ListFooterComponentStyle={{
    marginTop: 10,
  }} // Style for footer
  // --- Empty State ---
  ListEmptyComponent={<Empty />} // Rendered when data is empty
  // --- Separators ---
  ItemSeparatorComponent={() => <View style={{ height: 10 }} />} // Separator between items
  // --- Pull To Refresh ---
  refreshing={loading} // Controls refresh spinner
  onRefresh={handleRefresh} // Pull-to-refresh callback
  // --- Infinite Scrolling ---
  onEndReached={loadMore} // Triggered when near end
  onEndReachedThreshold={0.5} // Distance from end before triggering
  // --- Scrolling Behavior ---
  showsVerticalScrollIndicator={false} // Hide vertical scrollbar
  showsHorizontalScrollIndicator={false} // Hide horizontal scrollbar
  pagingEnabled={false} // Snap scrolling page-by-page
  snapToAlignment="start" // Snap alignment
  decelerationRate="fast" // Scroll speed behavior
  snapToInterval={300} // Snap every X pixels
  // --- Initial Rendering & Performance ---
  initialNumToRender={10} // Initial items rendered
  maxToRenderPerBatch={10} // Items rendered per batch
  windowSize={21} // Number of screens rendered outside viewport
  removeClippedSubviews={true} // Unmount offscreen views
  updateCellsBatchingPeriod={50} // Delay between render batches
  getItemLayout={(data, index) => ({
    length: 80,
    offset: 80 * index,
    index,
  })} // Optimization for fixed-height items
  // --- Re-render Control ---
  extraData={selectedId} // Forces FlatList re-render on external state change
  // --- Scroll Position ---
  initialScrollIndex={0} // Start from specific item
  scrollEnabled={true} // Enable/disable scrolling
  // --- Keyboard Handling ---
  keyboardShouldPersistTaps="handled" // Keyboard behavior on tap
  // --- Viewability Tracking ---
  onViewableItemsChanged={({ viewableItems, changed }) => {
    console.log(viewableItems);
  }} // Detect visible items
  viewabilityConfig={{
    itemVisiblePercentThreshold: 50,
  }} // Visibility rules
  // --- Multi Touch & Interaction ---
  disableVirtualization={false} // Disable virtualization (not recommended)
  bounces={true} // iOS bounce effect
  alwaysBounceVertical={false}
  alwaysBounceHorizontal={false}
  // --- Scroll Events ---
  onScroll={(event) => {
    console.log(event.nativeEvent.contentOffset.y);
  }} // Scroll listener
  scrollEventThrottle={16} // Scroll event frequency
  // --- Sticky Headers ---
  stickyHeaderIndices={[0]} // Sticky header indexes
  // --- Custom Refresh Control ---
  refreshControl={
    <RefreshControl refreshing={loading} onRefresh={handleRefresh} />
  }
  // --- Accessibility ---
  accessible={true}
  accessibilityLabel="User list"
  // --- Debugging ---
  debug={false} // Debug virtualization
/>
```
