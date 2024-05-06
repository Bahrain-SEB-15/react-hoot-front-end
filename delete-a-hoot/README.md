# ![React - Hoot Front-End - Delete a Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement the functionality for deleting a hoot.

## Overview

In this lesson, we’ll implement the following user story:

- As the author of a hoot, I should see a button to 'Delete' a hoot on the 'Details' page. Clicking on the button should delete the hoot, and redirect me back to the 'List' page.

When implementing delete functionality, it's important to ensure that **only the author of a given resource can delete it**. Our application should take measures to prevent unauthorized users from accessing this functionality.

These measures can be addressed in both the backend and frontend. In fact, we've already included a check for this in our **server**:

```js
// controllers/hoots.js

if (!hoot.author._id.equals(req.user._id)) {
  return res.status(403).send("You're not allowed to do that!");
}
```

In this lesson we will focus on restricting access on the **client-side**.

Based on our user story, we'll need to **conditionally render the delete button based authorship of the hoot**. Thankfully, we can accomplish this using the `AuthedUserContext` present in your React auth template. This makes the logged in `user` object easily accessible throughout our component tree. We'll make use of this `user` object when we render the delete button in `src/components/HootDetails/HootDetails.jsx`.

## Build the UI

At the top of `src/components/HootDetails/HootDetails.jsx`, import `AuthedUserContext` and `useContext`:

```jsx
// src/components/HootDetails/HootDetails.jsx

import { AuthedUserContext } from '../../App';
import { useState, useEffect, useContext } from 'react';
```

Within the component function, create the following `user` constant:

```jsx
// // src/components/HootDetails/HootDetails.jsx

const HootDetails = (props) => {
  const [hoot, setHoot] = useState(null);
  // Add the following
  const user = useContext(AuthedUserContext);
```

Time to add some conditional rendering for our button.

For our conditional rendering, we’ll make use of the [Logical AND ( && )](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND) operator.

If the `hoot.author._id` matches `user._id`, this piece of UI should be visible. If not, the UI should not be rendered. This means only the author of this particular `hoot` will be able to access the UI for updating or deleting a `Hoot`.

Add the following to `src/components/HootDetails/HootDetails.jsx`:

```jsx
// // src/components/HootDetails/HootDetails.jsx

<header>
  <p>{hoot.category.toUpperCase()}</p>
  <h1>{hoot.title}</h1>
  <p>
    {hoot.author.username} posted on
    {new Date(hoot.createdAt).toLocaleDateString()}
  </p>
  // Add the following:
  {hoot.author._id === user._id && (
    <>
      <button>Delete</button>
    </>
  )}
</header>
```

> 💡 Notice the use of a React fragment (`<>`) here. While we don't actually need a fragment at the moment, we'll be adding another element in the next lesson.

## Build the `handleDeleteHoot` function

Add the following to `src/App.jsx`:

```jsx
// src/App.jsx

const handleDeleteHoot = async (hootId) => {
  console.log('hootId', hootId);
};
```

Next, pass `handleDeleteHoot` down to `<HootDetails>`:

```jsx
// src/App.jsx

<Route
  path="/hoots/:hootId"
  element={<HootDetails handleDeleteHoot={handleDeleteHoot} />}
/>
```

Back in `src/components/HootDetails/HootDetails.jsx`, we can now update the delete button that we added in the previous section. We'll add an `onClick` event handler that calls upon `props.handleDeleteHoot(hootId)`.

Update your code as shown below:

```jsx
// src/components/HootDetails/HootDetails.jsx

<button onClick={() => props.handleDeleteHoot(hootId)}>Delete</button>
```

> 🚨 Be sure to pass in `hootId` as an argument when you call upon the function.

In your browser, try deleting a hoot. You should see that the `hootId` is being passed up the component tree.

With the `hootId` accessible in `handleDeleteHoot`, let's confirm that we can `filter()` state using this value:

```jsx
// src/App.jsx

const handleDeleteHoot = async (hootId) => {
  console.log('hootId', hootId);
  setHoots(hoots.filter((hoot) => hoot._id !== hootId));
  navigate('/hoots');
};
```

Remember, the [Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) method returns a shallow copy of the array, excluding all elements that do not pass the test implemented by the provided callback function.

In the code block above, our `filter()` method returns only the `hoot` objects whose `_id` values **do not match** the `hootId`.

Try deleting a hoot. After clicking the delete button, you should be directed to the list page, where the hoot is no longer present. If you refresh your browser, you'll notice that the hoot appears once more. This is occurs because at the moment, **we are only managing our local state**. No change has been made to the database. When the browser is refreshed, our `hootService.index()` runs once again, loading hoots from our database.

Managing local state is a great practice, in that it provides immediate visual updates for users. But for these changes to persist, state updates must be made in tandem with changes to the database. We'll address this issue in the next step!

## Build the service function

Let's finish up our delete functionality by adding the service.

Add the following to `src/services/hootService.js`:

```js
// src/services/hootService.js

const deleteHoot = async (hootId) => {
  try {
    const res = await fetch(`${BASE_URL}/${hootId}`, {
      method: 'DELETE',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
    });
    return res.json();
  } catch (error) {
    console.log(error);
  }
};

export {
  index,
  show,
  create,
  createComment,
  // Add export:
  deleteHoot,
};
```

## Call upon the service

Now that we have our service function, we'll add it to `handleDeleteHoot`, along with one other small change.

In our backend, you might recall that the delete hoot controller function responds with a `deletedHoot`:

```js
res.status(200).json(deletedHoot);
```

If we call upon `hootService.deleteHoot()`, what we get back is this `deletedHoot` object:

```jsx
const deletedHoot = await hootService.deleteHoot(hootId);
```

The `deletedHoot` contains the ObjectId (`_id`) of the hoot that was removed from our database. Knowing this, when we use the `filter()` method inside `handleDeleteHoot`, we can utilize the value of `deletedHoot._id` instead of the current `hootId`.

Doing so gives us additional assurance that the deletion was successful on the backend, before we make updates to our frontend.

Back in `src/App.jsx`, update `handleDeleteHoot` with the following:

```jsx
// src/App.jsx

const handleDeleteHoot = async (hootId) => {
  // Call upon the service function:
  const deletedHoot = await hootService.deleteHoot(hootId);
  // Filter state using deletedHoot._id:
  setHoots(hoots.filter((hoot) => hoot._id !== deletedHoot._id));
  // Redirect the user:
  navigate('/hoots');
};
```

Try it out! You should now be able to delete hoots.
