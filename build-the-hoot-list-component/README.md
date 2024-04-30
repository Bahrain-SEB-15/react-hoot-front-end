# ![React Hoot - Build the HootList](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to build a component that displays a list of hoots.

## Overview

In this lesson, we’ll implement the following user story:

- AAU, I should be able to see a list of all hoots on a 'List' page.

Let's walk through some of the logic involved here.

Our app will store `hoots` state in `src/App.jsx`. State will be passed down to the `src/components/HootList.jsx` component.

Within `HootList`, we’ll map through the `hoots` to produce an array of hoot `<article>` tags. Each `<article>` tag will be responsible for displaying a single `hoot` object.

The data held in `hoots` state will come from our backend. Retrieving the data on our frontend will require the use of the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). We'll group these `fetch()` requests in a dedicated module for each resource in our application. These modules are commonly referred to as **services**. 

## Scaffold the component

Run the following command in your terminal:

```bash
touch src/components/HootList.jsx
```

Let’s add some basic JSX scaffolding to the component. We'll include the component name in our `return` to help us verify that our navigation is working correctly in the next steps.

Add the following to `src/components/HootList/HootList.jsx`:

```jsx
// src/components/HootList/HootList.jsx
const HootList = (props) => {
  return (
    <main>
      Hoot List
    </main>
  )
}

export default HootList
```

Head over to `src/components/NavBar/NavBar.jsx`. We'll need to build the UI that allows a user to navigate to this component. While we are here, let's remove the welcome message, and update the text content of our dashboard `<Link>`.

Your authenticated `user` links should look like the following:

```jsx
      {user ?
        <ul>
          <li><Link to='/'>HOME</Link></li>
          <li><Link to='/hoots'>HOOTS</Link></li>

          <li><Link to='' onClick={handleLogout}>LOG OUT</Link></li>
        </ul>
```

In your browser, clicking on the 'HOOTS' link should now direct you to `/hoots`. Try it out. You might notice that the user interface remains unchanged.

To resolve this issue, we'll need to add a new `<Route>` in `src/App.jsx`.

First let's import the `HootList` component at the top of `src/App.jsx`:

```jsx
// src/App.jsx
import HootList from './components/HootList/HootList';
```

With the component imported, we are ready to build out the `<Route/>`.

This `<Route/>` and others like it will need to be *protected*, meaning they **can only be accessed by logged in users**. 

Protected routes can be implemented with a ternary, as seen in our application's starter code:

```jsx
// src/App.jsx
        {
          user ?
            <Route path='/' element={<Dashboard user={user} />} />
            :
            <Route path='/' element={<Landing />} />
        }
```

With the code snippet above, logged in users can access the protected `Dashboard` route. Users who are not logged in can only access the publicly available `Landing` page route. Our application will require several protected routes, so we'll need to make use of a React fragment (`<></>`) to group them together.

Update your protected routes in `**src/App.jsx**` with the following:

```jsx
// src/App.jsx
        <Routes>
          {user ?
            // Protected Routes:
            <>
              <Route path="/" element={<Dashboard user={user} />} />
              <Route path="/hoots" element={<HootList />} />
            </>
            :
            // Public Route:
            <Route path="/" element={<Landing />} />
          }
          <Route path="/signup" element={<SignupForm setUser={setUser} />} />
          <Route path="/signin" element={<SigninForm setUser={setUser} />} />
        </Routes>
```

With our `<Route>` in place, we should now be able to navigate to the `HootList` component.

## Add `index` functionality

The next step will be fetching data for the `HootList` to render. Utilizing the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), we'll create an asynchronous `index` service function that retrieves a list of hoots from our backend. 

### Create `hootService.js`

We'll group all services related to the hoot resource in a dedicated module called `hootService.js`. This pattern works well, as all hoot related service functions will make requests to the same `BASE_URL` endpoint on our server (`'/hoots'`). When a service function needs to make a request to a more precise endpoint, we'll modify the endpoint in the function itself.

Let's create our `hootService` module.

Run the following command in your terminal:

```bash
touch .env
touch src/services/hootService.js
```

And add the following to the top of `src/services/hootService.js`:

```js
const BASE_URL = `${import.meta.env.VITE_EXPRESS_BACKEND_URL}/hoots`
```

### Build the service function

Next we'll need to build out the `index` functionality. We'll be making a request to `'/hoots'`, so in this instance no modifications to the `BASE_URL` are necessary.

Add the following to `src/services/hootService.js`:

```js
// src/services/hootService.js
const index = async() => {
  try {
    const res = await fetch(BASE_URL, {
      headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` },
    })
    return res.json()
  } catch (error) {
    console.log(error)
  }
}

export { 
  index,
}
```

Notice the inclusion of the `headers` property. The `headers` property is an object with any headers to be sent alongside the request. In this case, we are including an `'Authorization'` header with a bearer token. This token is decoded by the `verifyToken` middleware function on our server, allowing us to indentify the logged in user, and ensuring that only a logged in user can access this functionality.

If you look at the `controllers/hoots.js` file in your backend application, you'll notice that all of our routes for hoots are **protected**, as they follow `verifyToken` in our middleware pipeline.

```js
// ========= Protected Routes =========
router.use(verifyToken);
// ... hoot routes/controllers
```

As a result, all of our hoot service functions will require this `'Authorization'` header.

> 🚨 Don't forget to `export` each service function after adding them. Otherwise they will not be accessible in the component where they are called upon.

### Call upon the service

Back in `src/App.jsx`, add an import for our new `hootService` module:

```jsx
// src/App.jsx
import * as hootService from './services/hootService';
```

> 💡 The syntax above is a great way to import everything (`*`) from the module. Within `src/App.jsx`, individual functions can be called upon with *dot notation* through the `hootService` object.

While we are here, let's import the `useEffect` hook as well:

```jsx
// src/App.jsx
import { useState, createContext, useEffect } from 'react';
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
      console.log('hootsData:', hootsData);
    };
    if (user) fetchAllHoots();
  }, [user]);
```

Notice the inclusion of `user` in our dependency array and the `if` condition placed around the invocation of `fetchAllHoots()`. 

Placing `user` in the dependency array causes the effect to fire off when the page loads or `user` state changes. Within our `useEffect`, we invoke `fetchAllHoots`, which in turn calls upon the `index` service. On the backend, our hoots `index` route is protected, which means **the request won’t go through until a user is logged in**. Including this `if` condition prevents the request from being made if a user is not logged in.

Check your browser console and verify that you are receiving `hootsData` from the backend.

After verifying the data, we should be ready to `setHoots` state:

```jsx
// src/App.jsx
  useEffect(() => {
    const fetchAllHoots = async () => {
      const hootsData = await hootService.index();

      // Set state:
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

Add the following to `src/components/HootList.jsx`:

```jsx
// src/components/HootList.jsx
const HootList = (props) => {
  return (
    <main>
      {props.hoots.map(hoot => (
        <p key={hoot._id}>
          {hoot.title}
        </p>
      ))}
    </main>
  )
}
```

Check your browser and click on the **Hoots** link. If you have existing hoots in your database you should now see a list of titles when you navigate to `/hoots`.

> 🚨 If you deleted all of the hoots in the database at the end of the Express REST API lesson, open up Postman and add a few new hoots so that you'll have data for this section of the lesson.

Time to touch up our JSX. We'll replace the existing `<p>` tags with clickable links that eventually navigate a user to an details page. First we'll need the `<Link>` component from `'react-router-dom'`.

Add the following import to `src/components/HootList/HootList.jsx`:

```jsx
// src/components/HootList/HootList.jsx
import { Link } from 'react-router-dom'
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
                {hoot.author.username}
                posted on
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