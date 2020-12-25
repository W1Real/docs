# Drag Overlay

The `<DragOverlay>` component provides a way to render a draggable overlay that is removed from the normal document flow and is positioned relative to the viewport.

![](../../.gitbook/assets/dragoverlay.png)

## When should I use a drag overlay?

Depending on your use-case, you may want to use a drag overlay rather than transforming the original draggable source element that is connected to the [`useDraggable`](usedraggable.md) hook:

* If you'd like to **show a preview** of where the draggable source will be when dropped, you can update the position of the draggable source while dragging without affecting the drag overlay.
* If your item needs to **move from one container to another while dragging**, we highly recommend you use the `<DragOverlay>` component so the draggable item can unmount from its original container while dragging and mount back into a different container without affecting the drag overlay.
* If your draggable item is within a **scrollable container,** we also recommend you use a `<DragOverlay>`, otherwise you'll need to set the draggable element to `position: fixed` yourself so the item isn't restricted to the overflow and stacking context of its scroll container, and can move without being affected by the scroll position of its container.
* If your `useDraggable` items are within a **virtualized list**, you will absolutely want to use a drag overlay, since the original drag source can unmount while dragging as the virtualized container is scrolled.
* If you want **smooth drop animations** without the effort of building them yourself.

## Usage

You may render any valid JSX within the children of the `<DragOverlay>`. However, **make sure that the components rendered within the drag overlay do not use the `useDraggable` hook**.  

The `<DragOverlay>` component should remain mounted at all times so that it can perform the drop animation. If you conditionally render the `<DragOverlay>` component, drop animations will not work.

Instead, you should conditionally render the children passed to the `<DragOverlay>`:

{% tabs %}
{% tab title="App.jsx" %}
```jsx
import React, {useState} from 'react';
import {DndContext, DragOverlay} from '@dnd-kit/core';

import {Draggable} from './Draggable';

/* The implementation details of <Item> and <ScrollableList> are not
 * relevant for this example and are therefore omitted. */

function App() {
  const [items] = useState(['1', '2', '3', '4', '5']);
  const [activeId, setActiveId] = useState(null);
  
  return (
    <DndContext onDragStart={handleDragStart} onDragEnd={handleDragEnd}>
      <ScrollableList>
        {items.map(id =>
          <Draggable key={id} id={id}>
            <Item value={`Item ${id}`} />
          </Draggable>
        )}
      </ScrollableList>
      
      <DragOverlay>
        {activeId ? (
          <Item value={`Item ${activeId}`} /> 
        ): null}
      </DragOverlay>
    </DndContext>
  );
  
  function handleDragStart(event) {
    setActiveId(event.active.id);
  }
  
  function handleDragEnd() {
    setActiveId(null);
  }
}
```
{% endtab %}

{% tab title="Draggable.jsx" %}
```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function Draggable(props) {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <li ref={setNodeRef} {...listeners} {...attributes}>
      {props.children}
    </li>
  );
}
```
{% endtab %}
{% endtabs %}

## Recommended patterns

### Presentational components with forwarded refs

We **highly** recommend that all the components you intend to make draggable be [presentational components ](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)that are decoupled from `@dnd-kit` entirely.

Use the[ ref forwarding pattern](https://reactjs.org/docs/forwarding-refs.html) to connect your presentational components to the `useDraggable` hook:

```jsx
import React, {forwardRef} from 'react';

const Item = forwardRef(({children, ...props}, ref) => {
  return (
    <li {...props} ref={ref}>{children}</li>
  )
});
```

This way, you can create two versions of your component, one that is presentational, and one that is draggable and renders the presentational component without the need for additional wrapper elements:

```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function DraggableItem(props) {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });
  
  return (
    <Item ref={setNodeRef} {...attributes} {...listeners}>
      {value}
    </Item>
  )
});
```

The advantage of this pattern is that you can then 

### Portals

The drag overlay is not rendered in a portal by default. Rather, it is rendered in the container where it is rendered. 

If you would like to render the `<DragOverlay>` in a different container than where it is rendered, import the [`createPortal`](https://reactjs.org/docs/portals.html) helper from `react-dom`:

```jsx
import React, {useState} from 'react';
import {createPortal} from 'react-dom';
import {DndContext, DragOverlay} from '@dnd-kit/core';

function App() {
  return (
    <DndContext>
      {createPortal(
        <DragOverlay>{/* ... */}</DragOverlay>,
        document.body,
      )}
    </DndContext>
  );
}
```


