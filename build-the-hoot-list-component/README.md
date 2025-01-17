# ![React - Hoot Front-End - Build the HootList Component](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to build a component that displays a list of hoots.

## Sign into the application

Before we begin building the "Hoot" pages for our application, let’s confirm that the authentication system is working properly. This will ensure that the React Auth Template is correctly connected to the Hoot Back-End API.

1. Open the application in your browser.
2. Use the sign-up form to create a new user account or sign in as an existing user (preferably with existing hoots!).

If everything is set up correctly you’ll be signed in and redirected to the `Dashboard` page.

## Building the HootList component

In this lesson, we’ll implement the following user story:

- As a User, I should be able to see a list of all hoots on a 'List' page.

Let's walk through some of the logic involved here:

- Our app will store `hoots` state in `src/App.jsx`. This state will be passed down to the `src/components/HootList.jsx` component.

- Within `HootList`, we’ll `map()` over `hoots` to produce an array of `<article>` tags. Each `<article>` tag will be responsible for rendering a single `hoot` object.

- The data held in `hoots` state will originate in our backend. Retrieving that data on the frontend will require the use of the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). We'll group these `fetch()` requests in a dedicated module for each resource in our application. These modules are commonly referred to as **services**.

## Scaffold the component

Let's get started!

Run the following commands in your terminal:

```bash
mkdir src/components/HootList
touch src/components/HootList/HootList.jsx
```

Let’s add some basic JSX scaffolding to the component. We'll include the component name in our `return` to help us verify that our navigation is working correctly in the next steps.

Add the following to `src/components/HootList/HootList.jsx`:

```jsx
// src/components/HootList/HootList.jsx

const HootList = (props) => {
  return <main>Hoot List</main>;
};

export default HootList;
```

Head over to `src/components/NavBar/NavBar.jsx`. We'll need to add a new link that allows a user to navigate to this component.

1. Add a link to the new `HootList` component at `/hoots`.

2. While we are here, let's remove the welcome message, and update the text content of our `Dashboard` `<Link>` to `HOME`.

Your authenticated `user` links should look like the following:

```jsx
// src/components/NavBar/NavBar.jsx

{user ? (
  <ul>
    <li><Link to="/">HOME</Link></li>
    <li><Link to="/hoots">HOOTS</Link></li>
    <li><Link to="/" onClick={handleSignOut}>Sign Out</Link></li>
  </ul>
```

Next we need to add a new `<Route>` to match this new `<Link>` in `src/App.jsx`.

## Add a Route for `HootList`

First navigate to `src/App.jsx` and import the `HootList` component near the top of the file:

```jsx
// src/App.jsx

import HootList from "./components/HootList/HootList";
```

With the component imported, we are ready to add the `<Route/>`.

Certain routes in our application, like the `HootList` page, should only be accessible to **logged-in users**. These are called **protected routes**.

We can implement protected routes using a ternary operator to check if a user is logged in. If the user exists, they gain access to the protected routes; otherwise, they are redirected or shown a placeholder (like a `404`).

```jsx
{ user ? (
    // Protected Routes:
    <>
      <Route path="/hoots" element={<HootList />} />
    </>
  ) : (
    // If no user is logged in, render an empty placeholder:
    <></>
  );
}
```

Our application will require several protected routes, so we'll need to make use of a React fragment (`<> </>`) to group them together.

Update your protected routes in `src/App.jsx` with the following:

```jsx
// src/App.jsx

<Routes>
  <Route path="/" element={user ? <Dashboard /> : <Landing />} />
  {user ? (
    <>
      <Route path="/hoots" element={<HootList />} />
    </>
  ) : (
    <></>
  )}
  <Route path="/sign-up" element={<SignUpForm />} />
  <Route path="/sign-in" element={<SignInForm />} />
</Routes>
```

> 💡 In React, ternary operators allow you to conditionally display different components or groups of components based on a specific condition. In the code snippet above, we can use a ternary to both conditionally render a specific element for the same path "/" or protect groups of routes based logged in user. 

With our `<Route>` in place, we should now be able to navigate to the `HootList` component.

## Create `hootService.js`

To display the list of hoots, we need to fetch the data from our backend. Here's an overiew of how we'll retrive our hoot data:

1. First, we'll set up a service module.

   - We'll group all services related to the hoot resource in a dedicated module called `hootService.js`.
   - This keeps our code organized and makes it easy to manage similar API calls.
   - All hoot-related service functions will use the same base URL `('/hoots')` for the server.
   - If a function needs a more specific endpoint like `'/hoots/:id'`, we’ll handle it directly within that function.

2. Next we'll need to fetch data for the `HootList` to render.

   - Utilizing the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), we'll create an asynchronous `index` service function that retrieves a list of hoots from our backend.

### Let's create our `hootService` module!

Run the following command in your terminal:

```bash
touch src/services/hootService.js
```

And add the following to the top of `src/services/hootService.js`:

```js
const BASE_URL = `${import.meta.env.VITE_BACK_END_SERVER_URL}/hoots`;
```

> 💡 If you completed the setup steps, your `.env` should contain a `VITE_BACK_END_SERVER_URL` environment variable set to `"http://localhost:3000"`. When running our app locally, the `BASE_URL` will read as `http://localhost:3000/hoots`.

## Build the service function

Next we'll need to build out the `index` functionality. We'll be making a request to `'/hoots'`, so in this instance no modifications to the `BASE_URL` are necessary.

Add the following to `src/services/hootService.js`:

```js
// src/services/hootService.js

const index = async () => {
  try {
    const res = await fetch(BASE_URL, {
      headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
    });
    return res.json();
  } catch (error) {
    console.log(error);
  }
};

export { index };
```

> 🚨 Don't forget to `export` each service function after adding them. Otherwise they will not be accessible in the component where they are called upon.

Notice the inclusion of the `headers` property. The `headers` property is an object containing any headers that need to be sent along with the request. In this case, we are including an `'Authorization'` header with a **bearer token**. This token is decoded by the `verifyToken` middleware function on our server, allowing us to indentify the logged in user, and ensuring that only a logged in user can access this functionality.

If you look at the `controllers/hoots.js` file in your backend application, you'll notice that all of our routes for hoots are **protected** by the `verifyToken` middleware..

```js
router.get('/', verifyToken, async (req, res) => {...}
```

As a result, all of our hoot service functions will require this `'Authorization'` header.

## Call upon the service

Back in `src/App.jsx`, add an import for our new `hootService` module:

```jsx
// src/App.jsx

import * as hootService from "./services/hootService";
```

> 💡 The syntax above is a great way to import everything (`*`) from the module. Within `src/App.jsx`, individual functions can be called upon with _dot notation_ through the `hootService` object.

While we are here, let's import the `useEffect` hook as well:

```jsx
// src/App.jsx

import { useContext, useState, useEffect } from "react";
```

Before we retrieve a list of hoots from our backend, we'll need a state variable to store them in.

Let's create a new `useState` variable called `hoots`:

```jsx
// src/App.jsx

const [hoots, setHoots] = useState([]);
```

Next, we'll use our effect to trigger our `index` service function. At the moment, we'll just verify that we are getting the data we need with `console.log()`.

Add the following:

```jsx
// src/App.jsx

useEffect(() => {
  const fetchAllHoots = async () => {
    const hootsData = await hootService.index();

    // console log to verify
    console.log("hootsData:", hootsData);
  };
  if (user) fetchAllHoots();
}, [user]);
```

Notice the inclusion of `user` in our dependency array and the `if` condition placed around the invocation of `fetchAllHoots()`.

> Placing `user` in the dependency array causes the effect to fire off when the page loads or `user` state changes. Within our `useEffect`, we invoke `fetchAllHoots`, which in turn calls upon the `index` service. On the backend, our hoots `index` route is protected, which means **the request won’t go through until a user is logged in**. Including this `if` condition prevents the request from being made if a user is not logged in.

1. Check your browser console and verify that you are receiving `hootsData` from the backend.

2. After verifying the data, we should be ready to `setHoots` state:

```jsx
// src/App.jsx

useEffect(() => {
  const fetchAllHoots = async () => {
    const hootsData = await hootService.index();

    // update to set state:
    setHoots(hootsData);
  };
  if (user) fetchAllHoots();
}, [user]);
```

Once state is set, we can pass `hoots` down to the `<HootList/>` component:

```jsx
// src/App.jsx

<Route path="/hoots" element={<HootList hoots={hoots} />} />
```

Within `src/components/HootList.jsx`, verify that `hoots` is accessible through `props`.

> 🏆 After passing props, it is best verify that the data is being passed down to the child component with your React Dev Tools or a `console.log`. Doing so will generally make rendering the data easier.

## Render a list of hoots

The next step is to `map()` over `props.hoots`. At this stage, we'll use the `Array.prototype.map()` method to produce an array of `<p>` tags before replacing these with a proper 'card' UI element.

Add the following to `src/components/HootList/HootList.jsx`:

```jsx
// src/components/HootList/HootList.jsx

const HootList = (props) => {
  return (
    <main>
      {props.hoots.map((hoot) => (
        <p key={hoot._id}>{hoot.title}</p>
      ))}
    </main>
  );
};
```

Check your browser and click on the **Hoots** link. If you have existing hoots in your database you should now see a list of titles when you navigate to `/hoots`.

> 🚨 If you deleted all of the hoots in the database at the end of the Express REST API lesson, open up Postman and add a few new hoots so that you'll have data for this section of the lesson.

Time to touch up our JSX. We'll replace the existing `<p>` tags with clickable links that eventually navigate a user to an details page. First we'll need the `<Link>` component from `'react-router-dom'`.

Add the following import to `src/components/HootList/HootList.jsx`:

```jsx
// src/components/HootList/HootList.jsx

import { Link } from "react-router";
```

And update the `return` with the following:

```jsx
// src/components/HootList/HootList.jsx

return (
  <main>
    {props.hoots.map((hoot) => (
      <Link key={hoot._id} to={`/hoots/${hoot._id}`}>
        <article>
          <header>
            <h2>{hoot.title}</h2>
            <p>
              {hoot.author.username} posted on
              {new Date(hoot.createdAt).toLocaleDateString()}
            </p>
          </header>
          <p>{hoot.text}</p>
        </article>
      </Link>
    ))}
  </main>
);
```

Notice how we are wrapping the `<article>` with a `Link` component. The `to` property specifies the URL a user should be directed to when the `Link` is clicked. Think of the value assigned to the `to` property as an argument passed into a function. Once we add params (`:hootId`) on a corresponding client side route, this `Link` will direct a user to a details page for a specific hoot whenever they click on a card.

Try clicking on on a hoot. You should be taken to a URL like the one below:

```plaintext
http://localhost:5173/hoots/660c468afc392b1ea00e9fa7
```

The next step will be building the interface to be displayed at this URL.
