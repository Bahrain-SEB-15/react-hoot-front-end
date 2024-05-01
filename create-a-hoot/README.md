# ![React - Hoot Front-End - Create a Hoot](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to build a component for creating new hoots.

## Overview

In this lesson, we’ll implement the following user story:

- AAU, I should be able to create a hoot post.

This will require a `<form>` component that allows users to create new hoots. Upon submitting a new hoot, the user should be redirected back to the 'List' page. 

To create a hoot, we'll make a `POST` request to our server. When a request is made, we'll use the response to update the `hoots` state held in `src/App.jsx`. This data will then flow down to `src/components/HootList/HootList.jsx`, where we will be able to see our newly added hoot.

## Scaffold the component

First let's add a new link to our navigation bar. It should direct users to `/hoots/new`.

Add the following to `src/components/NavBar/NavBar.jsx`:

```jsx
// src/components/NavBar/NavBar.jsx
          <li><Link to="/hoots/new">NEW HOOT</Link></li>
```

> 🚨 Make sure you add this to the protected set of links!

If you ever wish to test out your client-side routes *before* creating the component, you can define the `<Route />` and render a simple element, like in the example below:

```jsx
              <Route
                path="/hoots/new"
                element={<h1>New Hoot</h1>/>}
              />
```

> 💡 Notice how we are adopting [RESTful/Resourceful Routing Conventions](https://www.notion.so/RESTful-Resourceful-Routing-Conventions-a54d1ddc99ee4a0cbda331addc6d1f97?pvs=21) in our client side routes. This isn’t a requirement, but sticking to familiar conventions can be helpful when collaborating with other developers.

Next, let's create the component.

Run the following commands in your terminal:

```bash
mkdir src/components/HootForm
touch src/components/HootForm/HootForm.jsx
```

Add the following to `src/components/HootForm/HootForm.jsx`:

```jsx
// src/components/HootForm/HootForm.jsx
import { useState } from "react";

const HootForm = (props) => {
  const [formData, setFormData] = useState({
    title: '',
    text: '',
    category: 'News',
  });

  const handleChange = (evt) => {
    setFormData({ ...formData, [evt.target.name]: evt.target.value })
  };

  const handleSubmit = (evt) => {
    evt.preventDefault();
    console.log('formData', formData);
    // We'll update this function shortly...
  }

  return (
    <main>
      <form onSubmit={handleSubmit}>
        <label htmlFor="title-input">Title</label>
        <input
          required
          type="text"
          name="title"
          id="title-input"
          value={formData.title}
          onChange={handleChange}
        />
        <label htmlFor="text-input">Text</label>
				<textarea
          required
          type="text"
          name="text"
          id="text-input"
          value={formData.text}
          onChange={handleChange}
        />
        <label htmlFor="category-input">Category</label>
        <select
          required
          name="category"
          id="category-input"
          value={formData.category}
          onChange={handleChange}
        >
          <option value="News">News</option>
          <option value="Games">Games</option>
          <option value="Music">Music</option>
          <option value="Movies">Movies</option>
          <option value="Sports">Sports</option>
          <option value="Television">Television</option>
        </select>
        <button type="submit">SUBMIT</button>
      </form>
    </main>
  );
};

export default HootForm;
```

This component should be pretty similar to other forms you’ve seen in React, but let’s touch on one interesting detail. Take a look at the `<select>` tag. This is a good example of how you can handle select menus in React. Notice how we set the default value for this field in the initial state of `formData`. This varies a bit from other `input` fields in that we have a pre-defined `value` attribute on each `<option>` tag. If you are using an `enum` constraint in your `schema`, make sure these values match!

> ❓ Notice our `handleSubmit` function. Why do we need [e.preventDefault()](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) when we submit a `<form>` in React? What default behavior are we preventing [here](https://react.dev/learn/responding-to-events#preventing-default-behavior)?

Take a moment to verify that you can successfully change `formData` state. When you submit the form, you should only see a `console.log` of state, as we have not yet built out the logic to create a new hoot.

## Build the `handleAddHoot` function

To make our form fully functional, we'll need to circle back to `src/App.jsx`. Here, we'll build out a `handleAddHoot` function.

First let's import the `useNavigate()` hook from `react-router-dom`. This will allow us to redirect a user back to the hoot list page after submitting a new hoot.

Add the following import to the top of `src/App.jsx`:

```jsx
// src/App.jsx
import { Routes, Route, useNavigate } from 'react-router-dom';
```

Next, create a new instance of the `useNavigate()` hook within the component function:

```jsx
// src/App.jsx
  const navigate = useNavigate();
```

Add the following function:

```jsx
// src/App.jsx
  const handleAddHoot = async (hootFormData) => {
    console.log('hootFormData', hootFormData);
    navigate('/hoots');
  };
```

At this point, we'll just confirm that the `hootFormData` is being passed to the function, and that `useNavigate()` is functioning correctly.

With the function in place, update your protected routes with the following:

```jsx
              <Route
                path="/hoots/new"
                element={<HootForm handleAddHoot={handleAddHoot} />}
              />
```

Now that we are passing down `handleAddHoot` as props, we can finish building out the `handleSubmit` function in `src/components/HootForm/HootForm.jsx`:
```jsx
// src/components/HootForm/HootForm.jsx
  const handleSubmit = (evt) => {
    evt.preventDefault();
    props.handleAddHoot(formData);
  };
```

> 🚨 Be sure to pass in `formData` state when calling upon `handleAddHoot`. 

Verify that our `hootFormData` is being passed up the component tree to `src/App.jsx` correctly. You should also be redirected to the hoot list page upon submitting the form.

## Build the service function

Next we'll build out the `create` service function. This will differ from previous service functions in this code-along, as it will require a `POST` request method. When using the Fetch API to make `POST` requests, we'll need to include a few additional properties in our request:

- **`method`**: The `method` property specifies the method of our request. With the Fetch API, this property is necessary whenever making a request other than the default `GET`.

- **`body`**: The `body` property specifies the form data to include in the request. We'll make use of the `JSON.stringify()` method here. Check out this link for more info on the [JSON object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON).

- **`'Content-Type'`**: Within our `headers` object, we'll also need to specify the data type of the information included in the `body` property. In this case, we'll set it to `'application/json'`.

Let's add the service:

```js
// src/services/hootService.js
const create = async (hootFormData) => {
  try {
    const res = await fetch(BASE_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(hootFormData)
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
}
```

## Call upon the service

Back in `src/App.jsx`, update `handleAddHoot` with the service function:

```jsx
// src/App.jsx
  const handleAddHoot = async (hootFormData) => {
    const newHoot = await hootService.create(hootFormData);
    setHoots([newHoot, ...hoots]);
    navigate('/hoots');
  };
```

Notice how we `setHoots` state. The `newHoot` is being added to the **front of the array**, followed by a copy of the existing `hoots` in state. This means that on submit, the newest `hoot` entry will appear at the top of the page. This will match the behavior of our `index` functionality, which returns `hoots` in descending order, meaning the most recent `hoots` are ordered ahead of older ones. If we added `newHoot` to the end of the array when we `setHoots` state, the order of elements would shift whenever the page refreshed, as this would trigger the `index` service once again. 

Try it out in your browser. You should now be able to add new hoots.