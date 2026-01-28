# Creating a Draggable Box

## Description

Probably the most difficult task of this project will be creating the interface in which to allow the boxes to be moved and resized.

## Why not use an existing library

To give myself extra controllability in one space for resizing and dragging boxes -- and because this dependency is not compatible with React 19 -- I will do this from scratch, and will not use a library like [React Draggable](https://www.npmjs.com/package/react-draggable)

The [GitHub project](https://github.com/react-grid-layout/react-draggable/tree/master) may prove to be a useful point of reference, however.

## Retrieving Element Size

ref: [How to Retrieve the Element Size and Position with React getBoundingClientRect?](https://www.dhiwise.com/post/how-to-retrieve-element-size-with-react-getboundingclientrec)

The above article is useful for understanding how to retrieve `BoundingRect` elements in React.

This allows you to get the x/y coordinates, as well as top/bottom/sides of the boxes. This is essential for resizing and repositioning. 

> _"BoundingRect is more appropriate for layout purposes where the rendered size and position are important."_

`BoundingRect`'s are required for both the draggable box and the box it is contained inside of, as not only does the box need to be draggable, it also needs to be stopped if it hits the edges.

To get the `BoundingRect` I had to create a `useRef` on the box and set the rect `onMouseClick`, then this is set to state using `useState`

`onMouseDown` and state seems to be the lesser of the evils as the rect does not load in a way that makes it reliable as a standard variable. I have attempted to set the variable outside of the function but to no avail. 

There may be a solution to create its own hook reliably, but I have not found this.

See the example code below:
```typescript
export function GridWorkspace(props: GridWorkspaceProps) {
  const {mouseDown, setMouseDown} = props

  const gridRef = useRef<HTMLInputElement>(null)
  const [rect, setRect] = useState<DOMRect | null>(null)

  function handleMouseDown(): void {
    if (gridRef.current) {
      setRect(gridRef.current?.getBoundingClientRect())
    }

    setMouseDown(true)
  }

  return (
    <main id="grid-workspace" ref={gridRef} onMouseDown={handleMouseDown}>
      <EditableBox mouseDown={mouseDown} gridRect={rect}/> : <></>
    </main>
  )
}
```

note that `mouseDown` is tracked from the `App` component down to the `EditableBox` (resizable and draggable box) itself. This is because to ensure that the box is still set to draggable, even up the mouse goes out of the grid.

(I have, however, set it to stop being draggable if it goes out of the window.)

## Retrieving Mouse Position

In conjunction with the above, I also need to ensure that the mouse position is tracked at all times.

This has been done via a custom `useMousePosition` hook. This tracks your mouse whenever it's in your window.

```typescript
const useMousePosition = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const updatePosition = (event: MouseEvent ) => {
      setPosition({ x: event.clientX, y: event.clientY });
    };
    window.addEventListener("mousemove", updatePosition);
    return () => {
      window.removeEventListener("mousemove", updatePosition);
    };
  }, []);
  return position;
};
```

## Dragging the Box

There is a fairly hefty function that works to drag the box.

This function - `moveBoxWithinGridByAxis(<axis>, {event, mousePos, rect, gridRect, borderWidth})` - runs consistently, as long as the following conditions are met:

1. A `useEffect` is triggered when an `isDraggable` variable is `true`. This works with an `onMouseDown` inside of the `Editablebox`
2. `boxRef.current` has a value. I.e. the `rect` has been set for the `Editablebox`
3. The mouse is moving: `window.addEventListener("mousemove", updatePosition);`

As this is dragged, the `rect`'s state is updated

### moveBoxWithinGridByAxis

This function does a lot of the heavy lifting with the dragging.

It retrieves either the `x` or `y` axis and sets them simultaneously.

Not taking into account the checks to see whether it's hitting the edges, the box is made draggable with the following calculation:

```typescript
event[client<X|Y>] - mousePos[<x|y>] + rect[<x|y>] + window[scroll]
```

Let's break this down in steps:
1. `event[client<X|Y>]` - Constantly checks where the mouse is whilst being dragged
2. `mousePos[<x|y>]` - The mouses original position at the start of dragging
3. `rect[<x|y>]` - Where the boxes top and left sides were at the start of dragging
4. `window[scroll]` - Compensates far down/right the window is scrolled

So, essentially, this tracks the difference in your mouse position (`event[client<X|Y>] - mousePos[<x|y>]`), and changes your rect position (`+ rect[<x|y>]`), whilst accounting for the window being scrolled position (`+ window[scroll`)