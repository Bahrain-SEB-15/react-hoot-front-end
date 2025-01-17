# ![React - Hoot Front-End - Update a Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to implement the functionality for updating a hoot.

## Overview

In this lesson, we’ll implement the following user story:

- As the author of a hoot, I should see a link to 'Edit' a hoot on the 'Details' page. Clicking on the link should direct me to an 'Edit' page where I can modify the hoot. Upon submitting the update, I should be redirected back the the 'Details' page.

This functionality will **not require a new component**, as we will refactor the existing `src/components/HootForm/HootForm.jsx` to handle both **create** and **update**.

To accomplish this, we'll make use of the `useParams()` hook. The `useParams()` hook allows us to read data held in the URL, and based on that data, we'll augment the content and functionality our component.

## Build the UI

Before we modify our form, we'll add the 'Edit' `<Link>` that directs a user to that page.

Add the following import to the top of `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx

import { Link } from "react-router";
```

Next, add the `<Link>` directly above the 'Delete' `<button>`:

```jsx
// src/components/HootDetails/HootDetails.jsx

{
  hoot.author._id === user._id && (
    <>
      <Link to={`/hoots/${hootId}/edit`}>Edit</Link>

      <button onClick={() => props.handleDeleteHoot(hootId)}>Delete</button>
    </>
  );
}
```

> ❓ Why are we wrapping the `Link` and `button` elements in a [fragment](https://beta.reactjs.org/apis/react/Fragment#fragment)?

Take note of the value given to the `to` prop, it will be important in the following steps:

```js
`/hoots/${hootId}/edit`;
```

Add the following to your protected routes in `src/App.jsx`:

```jsx
// src/App.jsx

<Route path="/hoots/:hootId/edit" element={<HootForm />} />
```

In the next section, we'll access the value of this `hootId` parameter with the `useParams()` hook.

## Modify the `HootForm`

Head over to `src/components/HootForm/HootForm.jsx` and import `useParams` from `'react-route-dom'`:

```jsx
// src/components/HootForm/HootForm.jsx

import { useParams } from "react-router";
```

Within the component, call upon `useParams()` to access the `hootId`:

```jsx
// // src/components/HootForm/HootForm.jsx

const { hootId } = useParams();
```

With a `console.log`, verify that you can access the `hootId`.

We can also confirm this visually by adding an `<h1>` and a **ternary** to our `<form>`:

```jsx
// src/components/HootForm/HootForm.jsx

<main>
  <form onSubmit={handleSubmit}>
    <h1>{hootId ? 'Edit Hoot' : 'New Hoot'}</h1>
```

Now, if you navigate between the 'Edit' and 'New' pages, you should notice the title of the page changes, despite the same component being rendered.

This example demonstrates how we can modify other elements and behaviors of the component. If a `hootId` is present, we can assume the user has accessed the 'Edit' page, and requires update functionality. Otherwise, the basic 'New' form should be rendered along with our existing code for creating a hoot.

## Set `formData` state

The first modification we'll make to the functionality of the component relates to its initial state. If the user is updating a hoot, the inputs of our form should be prefilled with any existing hoot details. This will require calling upon the `hootService.show()` service within `src/components/HootForm/HootForm.jsx`.

At the top of `src/components/HootForm/HootForm.jsx`, add imports for `hootService` and `useEffect`:

```jsx
import { useState, useEffect } from "react";
import * as hootService from "../../services/hootService";
```

Add the following `useEffect()`

```jsx
// src/components/HootForm/HootForm.jsx

useEffect(() => {
  const fetchHoot = async () => {
    const hootData = await hootService.show(hootId);
    setFormData(hootData);
  };
  if (hootId) fetchHoot();
}, [hootId]);
```

Notice the `if` condition and the inclusion of `hootId` in the dependency array. If a `hootId` is present, we make a request to our server, and use the `hootData` response to `setFormData` state. If there is no `hootId`, we leave the initial state of `formData` unchanged.

Take a moment to confirm that the initial state of `formData` is being set correctly when editing a hoot.

## Build the `handleUpdateHoot` function

Next we'll add the `handleUpdateHoot` function in `src/App.jsx`

```jsx
// src/App.jsx

const handleUpdateHoot = async (hootId, hootFormData) => {
  console.log("hootId:", hootId, "hootFormData:", hootFormData);
  navigate(`/hoots/${hootId}`);
};
```

For now, we'll confirm that the function is receiving two pieces of data:

1. `hootId`

2. `hootFormData`

Next, pass the function down to the `<HootForm>`:

```jsx
// // src/App.jsx

<Route
  path="/hoots/:hootId/edit"
  element={<HootForm handleUpdateHoot={handleUpdateHoot} />}
/>
```

> 🚨 There are currently **two** routes rendering the `<HootForm>` in `src/App.jsx`. Be sure to pass `handleUpdateHoot` to the component being rendered for the `/hoots/:hootId/edit` route!

Back in `src/components/HootForm/HootForm.jsx`, make the following change to `handleSubmit`:

```jsx
// src/components/HootForm/HootForm.jsx

const handleSubmit = (evt) => {
  evt.preventDefault();
  if (hootId) {
    props.handleUpdateHoot(hootId, formData);
  } else {
    props.handleAddHoot(formData);
  }
};
```

Once again, we are relying on the `hootId` to determine the behavior of our component. If a `hootId` is present, we call upon `props.handleUpdateHoot(hootId, formData)`. Otherwise, we call upon `props.handleAddHoot(formData)`

Submit the edit form and confirm that the necessary data is being passed up the component tree.

## Build the service function

The following code should mirror much of the functionality you've seen elsewhere in this lesson. Our `update` service function will depart slightly from `create`, in that it issues a `PUT` request and requires `two` parameters. The first parameter will be used to identify the hoot, and the second parameter contains the information that the hoot will be updated with. Additionally, modifying `hoots` state with the updated hoot will be a bit more involved than what you saw with `handleAddHoot`.

Time to add the `update` service function:

```jsx
// src/services/hootService.js

async function update(hootId, hootFormData) {
  try {
    const res = await fetch(`${BASE_URL}/${hootId}`, {
      method: "PUT",
      headers: {
        Authorization: `Bearer ${localStorage.getItem("token")}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify(hootFormData),
    });
    return res.json();
  } catch (error) {
    console.log(error);
  }
}

export {
  index,
  show,
  create,
  createComment,
  deleteHoot,
  // As always, remember to export:
  update,
};
```

## Call upon the service

Next we'll update `handleUpdateHoot` with our service and set state accordingly.

Add the following to `src/App.jsx`:

```jsx
// src/App.jsx

const handleUpdateHoot = async (hootId, hootFormData) => {
  const updatedHoot = await hootService.update(hootId, hootFormData);

  setHoots(hoots.map((hoot) => (hootId === hoot._id ? updatedHoot : hoot)));

  navigate(`/hoots/${hootId}`);
};
```

This implementation of the [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) method is a bit different from the mapping of `JSX` elements you’ve seen in React previously. Let's take a moment to discuss the code above.

Remember, `hoots` state is an array of `hoot` objects. Calling upon `hootService.update()` has given us access to an `updatedHoot`. This `updatedHoot` object needs to be added to `hoots` state. To do so, we need to replace the original version of that object with the `updatedHoot`.

By mapping over the `hoots` array, we are able to check each `hoot` object. If the current element being processed has an `_id` that matches `updatedHoot._id`, we replace it with the `updatedHoot` that was returned from our backend. If the `_id` instances do not match, we simply return the existing element.

Through this process we are able to update a single object held in `hoots` state, while also maintaining an accurate record of the remaining elements in the array.

> 💡 If you are curious as to why something like the `splice()` method is not applicable here, check out React documentation on [updating arrays without mutation](https://react.dev/learn/updating-arrays-in-state#updating-arrays-without-mutation).

Try it out! After submitting a hoot for update, you should be directed to the list page with the modified hoot information present.
