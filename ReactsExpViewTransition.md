---

## Overview of `<ViewTransition>` in React

`<ViewTransition>` is an experimental feature in React that enables developers to add smooth animations to UI transitions within their applications. This component leverages the browser's View Transitions API to facilitate animations like cross-fades and shared element transitions during state changes, navigation, or DOM updates. As an experimental feature, it is not yet available in stable React versions and requires the use of experimental packages (`react@experimental`, `react-dom@experimental`, `eslint-plugin-react-hooks@experimental`). Note that these versions may contain bugs and are not recommended for production use[1].

## Key Features

- **Animation Triggers**: `<ViewTransition>` activates animations based on specific conditions such as entering the DOM (`enter`), exiting the DOM (`exit`), updating content (`update`), or transitioning between shared elements (`share`)[1].
- **Default Behavior**: By default, it applies a smooth cross-fade animation using the browser's built-in capabilities[1].
- **Customization**: Developers can customize animations by assigning CSS classes or objects to different transition types via props like `enter`, `exit`, `update`, `share`, and `default`[1].
- **Integration with Transitions**: Works seamlessly with React's `startTransition` to ensure non-blocking updates and smooth animations[2][3].

## Installation and Setup

Since `<ViewTransition>` is experimental, you must upgrade to the latest experimental versions of React packages to use it. Install them using:

```bash
npm install react@experimental react-dom@experimental eslint-plugin-react-hooks@experimental
```

Ensure your project is set up to handle experimental features, and avoid using them in production environments due to potential instability[1].

## Usage Examples

### Basic Animation on Enter/Exit

To animate an element when it enters or exits the DOM, wrap it with `<ViewTransition>` and trigger the state change within a `startTransition`:

```jsx
import { unstable_ViewTransition as ViewTransition, useState, startTransition } from 'react';

function Item() {
  return (
    <ViewTransition>
      <div>Hi</div>
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button onClick={() => {
        startTransition(() => {
          setShowItem((prev) => !prev);
        });
      }}>
        {showItem ? '➖' : '➕'}
      </button>
      {showItem ? <Item /> : null}
    </>
  );
}
```

In this example, the `enter` animation triggers when `showItem` becomes `true`, and the `exit` animation triggers when it becomes `false`. Note that `<ViewTransition>` must be placed before any DOM nodes to activate[1].

### Shared Element Transition

For animations between different components (e.g., from a thumbnail to a full-screen view), assign a unique `name` to `<ViewTransition>` to enable a shared element transition:

```jsx
import { unstable_ViewTransition as ViewTransition, useState, startTransition } from 'react';
import { Video, FullscreenVideo } from './Video';
import videos from './data';

export default function Component() {
  const [fullscreen, setFullscreen] = useState(false);
  if (fullscreen) {
    return (
      <FullscreenVideo 
        video={videos} 
        onExit={() => startTransition(() => setFullscreen(false))} 
      />
    );
  }
  return (
    <Video 
      video={videos} 
      onClick={() => startTransition(() => setFullscreen(true))} 
    />
  );
}
```

Ensure the `name` prop is unique across the app to avoid conflicts. Shared transitions take precedence over `enter`/`exit` animations if applicable[1].

### Animating List Reordering

To animate the reordering of items in a list, use `<ViewTransition>` around each item with a unique `key`:

```jsx
import { unstable_ViewTransition as ViewTransition, useState, startTransition } from 'react';
import { Video } from './Video';
import videos from './data';

export default function Component() {
  const [orderedVideos, setOrderedVideos] = useState(videos);
  const reorder = () => {
    startTransition(() => {
      setOrderedVideos((prev) => [...prev.sort(() => Math.random() - 0.5)]);
    });
  };
  return (
    <>
      <button onClick={reorder}>🎲</button>
      <div className="listContainer">
        {orderedVideos.map((video) => (
          <ViewTransition key={video.title}>
            <Video video={video} />
          </ViewTransition>
        ))}
      </div>
    </>
  );
}
```

This triggers an `update` animation for each item during reordering, provided `<ViewTransition>` is directly around the content without additional wrapper elements[1].

### Customizing Animations

Customize animations by specifying CSS classes for different transition types using props:

```jsx
import { unstable_ViewTransition as ViewTransition, useState, startTransition } from 'react';
import { Video } from './Video';
import videos from './data';

function Item() {
  return (
    <ViewTransition enter="slide-in" exit="slide-out" default="slow-fade">
      <Video video={videos} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button onClick={() => {
        startTransition(() => {
          setShowItem((prev) => !prev);
        });
      }}>
        {showItem ? '➖' : '➕'}
      </button>
      {showItem ? <Item /> : null}
    </>
  );
}
```

Define the animations in CSS using view transition pseudo-selectors:

```css
::view-transition-old(.slow-fade) {
  animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
  animation-duration: 500ms;
}

::view-transition-old(.slide-in) {
  animation: slide-out-left 0.3s ease-out;
}

::view-transition-new(.slide-in) {
  animation: slide-in-right 0.3s ease-out;
}
```

This allows tailored animations for each activation type[1][2].

## Props

`<ViewTransition>` supports several optional props to customize behavior and animations:

- **enter**: Defines the CSS class or object for animations when the component enters the DOM[1].
- **exit**: Specifies the class or object for animations when the component exits the DOM[1].
- **update**: Sets the class or object for animations during content updates[1].
- **share**: Configures the class or object for shared element transitions[1].
- **default**: Provides a fallback class or object for animations when no specific trigger matches[1].
- **name**: Assigns a unique identifier for shared element transitions; if omitted, React generates a unique name[1].

## Callbacks

Callbacks allow for programmatic control over animations, receiving the animated DOM `element` and `types` of transitions as arguments:

- **onEnter**: Called after an `enter` animation[1].
- **onExit**: Called after an `exit` animation[1].
- **onShare**: Called after a `share` animation[1].
- **onUpdate**: Called after an `update` animation[1].

## Integration with Other React Features

- **Suspense**: `<ViewTransition>` can animate content reveals within `<Suspense>` boundaries, treating transitions as `update` or `enter`/`exit` based on placement[1][5].
- **useTransition**: Works with `useTransition` to ensure non-blocking updates, activating animations only within `startTransition` calls[1][3].
- **addTransitionType**: Allows adding custom transition types for more granular animation control, useful in navigation scenarios[2][4].

## Caveats and Limitations

- **Experimental Status**: Not stable and unsuitable for production due to potential bugs[1].
- **DOM Restriction**: Currently only supported in DOM environments, with plans for React Native support in the future[1].
- **Placement Requirement**: Must be placed before any DOM nodes to activate animations; otherwise, no animation triggers[1].
- **User Preferences**: Does not automatically respect `prefers-reduced-motion`; developers must handle this via media queries[1].
- **Navigation API**: Animations may be skipped during legacy `popstate` events (e.g., browser back button); upgrading routers to the Navigation API resolves this[1].

## Troubleshooting

- **Animation Not Activating**: Ensure `<ViewTransition>` is placed before any DOM nodes in the component tree. Wrapping it inside other elements prevents activation[1].
- **Duplicate Name Error**: Avoid mounting multiple `<ViewTransition>` components with the same `name` simultaneously. Use unique identifiers or namespaces (e.g., `item-${id}`)[1].

## Conclusion

`<ViewTransition>` offers a powerful way to enhance user experience with animations in React applications, supporting a range of transition types and customization options. While its experimental nature limits its use to non-production environments, it provides a glimpse into future capabilities for seamless UI transitions. Developers should experiment with caution, ensuring proper setup and adherence to placement rules to leverage its full potential[1][2].

Citations:
[1] [<ViewTransition> – React](https://react.dev/reference/react/ViewTransition)  
[2] [React Labs: View Transitions, Activity, and more](https://react.dev/blog/2025/04/23/react-labs-view-transitions-activity-and-more)  
[3] [useTransition](https://react.dev/reference/react/useTransition)  
[4] [unstable_addTransitionType](https://react.dev/reference/react/addTransitionType)  
[5] [<Suspense> – React](https://react.dev/reference/react/Suspense)  
[6] [Experimenting with React View Transitions - Frontend at Scale](https://frontendatscale.com/issues/43/)  
[7] [useViewTransitionState | React Router API Reference](https://reactrouter.com/en/main/hooks/use-view-transition-state)  
[8] [React v18.0](https://react.dev/blog/2022/03/29/react-v18)  
[9] [View Transition API - MDN Web Docs - Mozilla](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)  
[10] [What You Need to Know About View Transitions in React - The Miners](https://blog.codeminer42.com/what-you-need-to-know-about-view-transitions-in-react/)  
[11] [useDeferredValue](https://react.dev/reference/react/useDeferredValue)  
[12] [Revealed: React's experimental animations API - Motion Blog](https://motion.dev/blog/reacts-experimental-view-transition-api)  
[13] [ViewTransitionOptions type | TanStack Router React Docs](https://tanstack.com/router/latest/docs/framework/react/api/router/ViewTransitionOptionsType)  
[14] [Common components (e.g. <div>)](https://react.dev/reference/react-dom/components/common)  
[15] [Using View Transition API in React App - Malcolm Kee](https://malcolmkee.com/blog/view-transition-api-in-react-app/)  
[16] [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)  
[17] [The View Transitions API And Delightful UI Animations (Part 2)](https://www.smashingmagazine.com/2024/01/view-transitions-api-ui-animations-part2/)  
[18] [viewTransition - next.config.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/viewTransition)  
[19] [ViewTransition - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/ViewTransition)  
[20] [React v19](https://react.dev/blog/2024/12/05/react-19)