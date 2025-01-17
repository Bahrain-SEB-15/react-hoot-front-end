# ![React - Hoot Front-End - Build the Hoot Details Component](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able build a component that renders details on a specific hoot.

## Overview

In this lesson, we’ll implement the following user story:

- As a User, clicking on a hoot in the 'List' page should navigate me to a 'Details' page where I can view information about a single hoot post along with its associated comments.

Our 'Details' page will be represented by `src/components/HootDetails/HootDetails.jsx`. This component will be responsible for rendering the details of a single hoot, including its associated comments. This component will be displayed whenever a user clicks on a hoot from the 'List' page.

Rendering details on a specific hoot will require a new service function to `fetch` a single `hoot` from our backend. For the service function to work, we’ll need to provide it with a `hoot._id` so that the appropriate hoot can be retrieved.

Our details component will differ from `src/components/HootList/HootList.jsx`, in that data will be held within the component's local state, as opposed to being stored in `src/App.jsx` and passed down as props.

## Scaffold the component

Let's build out the scaffolding for our component.

Run the following commands in your terminal:

```bash
mkdir src/components/HootDetails
touch src/components/HootDetails/HootDetails.jsx
```

Add the following to `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx

const HootDetails = (props) => {
  return <main>Hoot Details</main>;
};

export default HootDetails;
```

Next, `import` the component in `src/App.jsx`:

```jsx
// src/App.jsx

import HootDetails from './components/HootDetails/HootDetails';
```

And add the following **protected** route:

```jsx
// src/App.jsx

<Route
  path='/hoots/:hootId'
  element={<HootDetails />}
/>
```

With the addition of this client-side route, users should now be able to navigate to the `HootDetails` page by clicking on a hoot from the list page.

## Add `useParams`

When a user navigates to the `HootDetails` page, we'll need to `fetch()` details on that hoot. An individual hoot can be identified by its `Objectid`, with this value being accessible through the `hootId` parameter as defined on the `<Route>` above.

If `hootId` is our **parameter**, you might wonder where our **argument** is being passed in. Recall the `<Link>` we wrapped around our hoot cards. We gave it a `to` property set to `/hoots/${hoot._id}`.

This is where the actual `_id` data is passed in:

```jsx
// src/components/HootList/HootList.jsx

<Link key={hoot._id} to={`/hoots/${hoot._id}`}>
```

To extract this value for use in our component, we'll make use of the `useParams()` hook.

Add the following import to `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx

import { useParams } from 'react-router';
```

Next, let's call `useParams()` to get access to the `hootId`:

```jsx
// src/components/HootDetails/HootDetails.jsx

const HootDetails = (props) => {
  const { hootId } = useParams();
  console.log('hootId', hootId);

  return <main>Hoot Details</main>;
};
```

> 💡 Be sure to [destructure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) the `hootId` when calling upon `useParams()`!

Confirm that you have access to the `hootId` in `src/components/HootDetails/HootDetails.jsx`.

Now that we have the `hootId`, we should be able to retrieve details for that hoot from out backend using a new service function.

## Build the service function

Once again, our service function will require an Authorization header.

Add the following to `src/services/hootService.js`:

```js
// src/services/hootService.js

const show = async (hootId) => {
  try {
    const res = await fetch(`${BASE_URL}/${hootId}`, {
      headers: { Authorization: `Bearer ${localStorage.getItem('token')}` },
    });
    return res.json();
  } catch (error) {
    console.log(error);
  }
};

export {
  index,
  // Don't forget to export:
  show,
};
```

> ❓ Let’s take a moment to connect the dots of our application. Notice the `hootId` in the above service function. Where will this information be used in our backend?

## Call the service

Next up, we'll Call the service, and store the response from the server in state.

We'll need a few imports in `src/components/HootDetails/HootDetails.jsx` to proceed:

```jsx
// src/components/HootDetails/HootDetails.jsx

import { useState, useEffect } from 'react';
import * as hootService from '../../services/hootService';
```

Create a new `useState()` variable called `hoot` with an initial value of `null`:

```jsx
// src/components/HootDetails/HootDetails.jsx

const [hoot, setHoot] = useState(null);
```

> 💡 Giving `hoot` state an initial value of `null` will simplify some conditional rendering that we will implement shortly.

And add the following `useEffect()`:

```jsx
// src/components/HootDetails/HootDetails.jsx

useEffect(() => {
  const fetchHoot = async () => {
    const hootData = await hootService.show(hootId);
    setHoot(hootData);
  };
  fetchHoot();
}, [hootId]);

// Verify that hoot state is being set correctly:
console.log('hoot state:', hoot);
```

> 💡 Remember to include `hootId` in the [dependency array](https://beta.reactjs.org/apis/react/useEffect#specifying-reactive-dependencies) of your `useEffect()`. This tells the `useEffect()` to fire off whenever the value of the `hootId` changes.

Take a moment to confirm that `hoot` state is being set correctly. You should notice that the `author` property of a `hoot` is being populated.

## Render hoot details

If you included the `console.log()` in the step above, you might notice that the `hoot` state is `null` when the component first mounts. This can cause some issues if we try to render data that is not yet present in the component. Let's add a condition to account for that.

Add the following directly above your existing `return`:

```jsx
// src/components/HootDetails/HootDetails.jsx

if (!hoot) return <main>Loading...</main>;
```

With our condition in place, let's build out the remaining JSX:

```jsx
// src/components/HootDetails/HootDetails.jsx

return (
  <main>
    <header>
      <p>{hoot.category.toUpperCase()}</p>
      <h1>{hoot.title}</h1>
      <p>
        {hoot.author.username} posted on
        {new Date(hoot.createdAt).toLocaleDateString()}
      </p>
    </header>
    <p>{hoot.text}</p>
    <section>
      <h2>Comments</h2>
    </section>
  </main>
);
```

Notice the `<section>` tag at the bottom. This will act as our 'Comments' section. The `commentSchema` is embedded within `hootSchema`, so the relevant `comment` data should already exist within this component’s `hoot` state.

## Display comments

To display a hoot's associated comments, we'll want to `map()` over `hoot.comments` and produce a list of `<article>` tags.

Each comment's `<article>` tag should include a few things:

- The `username` of the comment's `author`.
- The `createdAt` date property of the the `comment`.
- The `text` content of the `comment`.

Regarding the `author` property of a `comment`, you might recall that our `show` controller on the backend is already populating the `author` information for each `comment`:

```js
// controllers/hoots.js

const hoot = await Hoot.findById(req.params.hootId).populate([
  'author',
  'comments.author',
]);
```

In our comments section, we'll also want to include a condition that displays a message if there are not yet any `comments` embedded within the `hoot`.

Update `src/components/HootDetails/HootDetails.jsx` with the following:

```jsx
// src/components/HootDetails/HootDetails.jsx

<section>
  <h2>Comments</h2>

  {!hoot.comments.length && <p>There are no comments.</p>}

  {hoot.comments.map((comment) => (
    <article key={comment._id}>
      <header>
        <p>
          {comment.author.username} posted on
          {new Date(comment.createdAt).toLocaleDateString()}
        </p>
      </header>
      <p>{comment.text}</p>
    </article>
  ))}
</section>
```

Check your browser. If you have any existing comments associated with a hoot, you should be able to see them now.
