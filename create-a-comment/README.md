# ![React - Hoot Front-End - Create a Comment](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be build a component for creating comments embedded within a hoot.

## Overview

In this lesson, we’ll implement the following user story:

- AAU, I should be able to add a comment on a hoot 'Details' page.

The user story will require a new component called `src/components/CommentForm/CommentForm.jsx`. It will mirror the functionality for creating hoots with one key difference. When creating comments, our `CommentForm` component **will not be a distinct page**, instead it will exist as a child component within `src/components/HootDetails/HootDetails.jsx`. Additionally, when updating state with a new comment, we'll need to modify `hoot` state, as this is the object where the `comments` array will reside. 

This lesson will serve as a good example of how to handle creating an embedded resource in a React app.

## Scaffold the component

Let's build out the scaffolding for our component.

Run the following commands in your terminal:

```bash
mkdir src/components/CommentForm
touch src/components/CommentForm/CommentForm.jsx
```

Add the following to `src/components/CommentForm/CommentForm.jsx`:

```jsx
// src/components/CommentForm/CommentForm.jsx
import { useState, useEffect } from "react"


import * as hootService from "../../services/hootService"

const CommentForm = (props) => {
  const [formData, setFormData] = useState({ text: '' })

  const handleChange = (evt) => {
    setFormData({ ...formData, [evt.target.name]: evt.target.value })
  }

  const handleSubmit = (evt) => {
    evt.preventDefault()
    // handleAddComment
    setFormData({ text: '' })
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="text-input">Your comment:</label>
      <textarea
        required
        type="text"
        name="text"
        id="text-input"
        value={formData.text}
        onChange={handleChange}
      />
      <button type="submit">SUBMIT COMMENT</button>
    </form>
  )
}

export default CommentForm
```

> 💡 Notice how we reset `formData` in our `handleSubmit` function. This is an important step, as we aren't navigating the user away from this page when a new comment is submitted.

Next, import the component at the top of `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx
import CommentForm from '../CommentForm/CommentForm';
```

Add the component to the comments section as shown below:

```jsx
// src/components/HootDetails/HootDetails.jsx
        <h2>Comments</h2>
        <CommentForm />
```

In your browser, verify that typing in the `CommentForm` updates `formData` correctly.

## Build the `handleAddComment` function

Next, let's create a `handleAddComment` function.

Add the following to `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx
  const handleAddComment = async (commentFormData) => {
    console.log('commentFormData', commentFormData);
  };
```

With the function in place, pass it down to the `<CommentForm />`:

```jsx
// src/components/HootDetails/HootDetails.jsx
        <CommentForm handleAddComment={handleAddComment} />
```

And update `handleSubmit` by calling upon `props.handleAddComment(formData)`:

```jsx
// src/components/CommentForm/CommentForm.jsx
  const handleSubmit = (evt) => {
    evt.preventDefault()
    props.handleAddComment(formData)
    setFormData({ text: '' })
  }
```

Confirm you are passing `formData` up to `src/components/HootDetails/HootDetails.jsx`.

## Build the service function

Time to build out the service function. Despite being another resouce, our comment service functions will live inside `src/services/hootService.js`. This is because all of the endpoints for comments will share the same `BASE_URL` as hoots (`'/hoots'`). We'll append more specific endpoints to each comment service function as necessary.

Add the following to `src/services/hootService.js`:

```js
// src/services/hootService.js
const createComment = async (hootId, commentFormData) => {
  try {
    const res = await fetch(`${BASE_URL}/${hootId}/comments`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(commentFormData)
    })
    return res.json()
  } catch (error) {
    console.log(error)
  }
}
```

## Call upon the service

With the service in place, we can complete the `handleAddComment` function in `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx
  const handleAddComment = async (commentFormData) => {
    const newComment = await hootService.createComment(hootId, commentFormData);
    setHoot({ ...hoot, comments: [...hoot.comments, newComment] });
  };
```

There is a lot going on in this example of `setHoot`. Let’s break it down.

We are storing a single `hoot` object in `hoot` state, but we need to update a particular property of this object (`hoot.comments`). 

First, we use the [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) to copy all existing properties of the `hoot` object.  

Within the `hoot` object, there is a `comments` property containing an array of `comment` objects. This array is the property we need to add our `newComment` to. To do so, we copy the existing `hoot.comments` array (again, using the spread operator), include the `newComment` at the end of the array, and finally assign this array to the `comments` property of the `hoot`.

Take a look at the code block below for a step by step breakdown:
    
```jsx
// Set state to an empty object:
setHoot({ })

// Set state to an object that includes all properties currently in Hoot state:
setHoot({ ...hoot })

// Much like the step above, except now the comments property of the object
// being set to state has its value set to an empty array:
setHoot({ ...hoot, comments: [ ] })

// Now the comments property of the object will include a copy of all 
// the comments that already exist in hoot state.
setHoot({ ...hoot, comments: [...hoot.comments] })

// And finally, we include the newComment at the end of the array:
setHoot({ ...hoot, comments: [...hoot.comments, newComment] })
```
    
Try it out in your browser. You should now be able to add comments!