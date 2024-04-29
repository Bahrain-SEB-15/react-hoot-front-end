# ![React Hoot - Building a Reusable Metadata Component](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to build a reusable metadata component.

## Overview

In this lesson we'll build a reusable metadata component. Metadata, in the context of online content such as a blog posts or comments, refers to data that provides information about other data. In regards to authorship, metadata typically includes details like the name of the author and the creation date.

Throughout our application, we are currently rendering this information with a `<p>` tag:

```jsx
        <p>
          {hoot.author.username} posted on{' '}
          {new Date(hoot.createdAt).toLocaleDateString()}
        </p>
```

Our `AuthorInfo` component will replace this `<p>` tag, with a more refined layout and styling. This makes it a bit easier to display information about an author in a visually consistent manner. 

Both `hoots` and `comments` will be able to make use of the `AuthorInfo` component. As a result, `AuthorInfo` is built to receive a generic `content` prop, so as not to mislabel either of the resources. 

The component will also display the date on which the resource was created, and a `ProfileIcon` representing the author.

## Build the component

Run the following commands in your terminal:

```bash
mkdir src/components/AuthorInfo
touch src/components/AuthorInfo/AuthorInfo.jsx
touch src/components/AuthorInfo/AuthorInfo.module.css
```

Add the following to `src/components/AuthorInfo/AuthorInfo.module.css`:

```css
.container {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.container * {
  margin: 0;
}

.container div {
  margin: 0 !important; 
}

.container > img {
  width: 30px;
  height: 30px;
  object-fit: cover;
  margin-right: 12px;
  border-radius: 50%;
  background-color: var(--background);
}

.container section > p {
  opacity: .75;
  line-height: 1;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 2px;
}

.container div p {
  line-height: .8;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 1px;
}

.container section {
  display: flex;
  flex-direction: column;
}

.container section img {
  opacity: .5;
  width: 14px;
  height: 14px;
  background: none;
  margin: 0 2px 0 -2px;
}
```

Add the following to `src/components/AuthorInfo/AuthorInfo.jsx`:

```jsx
// src/components/AuthorInfo/AuthorInfo.jsx
import styles from './AuthorInfo.module.css'
import ProfileIcon from '../../assets/profile.png';
import Icon from '../Icon/Icon'

const AuthorInfo = ({ content }) => {
  return (
    <div className={styles.container}>
      <img src={ProfileIcon} alt="The user's avatar" />
      <section>
        <p>{content.author.username}</p>
        <div className={styles.container}>
          <Icon category="Calendar" />
          <p>{new Date(content.createdAt).toLocaleDateString()}</p>
        </div>
      </section>
    </div>
  );
};

export default AuthorInfo;
```

> 💡 Because our application does not include photo upload, we'll make use of a generic `ProfileIcon` SVG.

## Apply the metadata component

Add the following import to `src/components/HootList/HootList.jsx`:

```jsx
import AuthorInfo from '../../components/AuthorInfo/AuthorInfo';
```

In `src/components/HootList/HootList.jsx`, locate the following `<p>` tag:

```jsx
// src/components/HootList/HootList.jsx

              <p>
                {hoot.author.username}
                posted on
                {new Date(hoot.createdAt).toLocaleDateString()}
              </p>

            </header>
            <p>{hoot.text}</p>
```

Replace this tag with the `<AuthorInfo />` component, passing down `content={hoot}`:

```jsx
// src/components/HootList/HootList.jsx

              <AuthorInfo content={hoot} />
            </header>
            <p>{hoot.text}</p>
```

Add the following import to `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx
import AuthorInfo from '../../components/AuthorInfo/AuthorInfo';
```

Locate the existing `<p>` tag in `src/components/HootDetails/HootDetails.jsx`:

```jsx
// src/components/HootDetails/HootDetails.jsx
        <p>
          {hoot.author.username} posted on{' '}
          {new Date(hoot.createdAt).toLocaleDateString()}
        </p>
```

And replace this tag with the `<AuthorInfo />` component, passing down `content={hoot}`:

```jsx
// src/components/HootDetails/HootDetails.jsx
      <header>
        <p>{hoot.category.toUpperCase()}</p>
        <h1>{hoot.title}</h1>

        <AuthorInfo content={hoot} />

        {hoot.author._id === user._id && (
          <>
            <Link to={`/hoots/${hootId}/edit`}>Edit</Link>
            <button onClick={() => props.handleDeleteHoot(hootId)}>
              Delete
            </button>
          </>
        )}

      </header>
```

Notice how we are labelling `hoot` as `content` when passing props to `<AuthorInfo>`. We do this because we'll be reusing `<AuthorInfo>` for our comments as well. Thankfully, the shape of a `hoot` and a `comment` are similar enough that we don't need to adjust any code inside `src/components/AuthorInfo/AuthorInfo.jsx`. By mapping `hoot` and a `comment` to a generic `content` prop, we avoid misrepresenting the data type or source being used in the component.

Next, we can add the `<AuthorInfo />` component to our list of comments, replacing the existing `<p>` tag.

Update `src/components/HootDetails/HootDetails.jsx` as shown below:

```jsx
// src/components/HootDetails/HootDetails.jsx
        {hoot.comments.map((comment) => (
          <article key={comment._id}>
            <header>

              <AuthorInfo content={comment} />

              {comment.author._id === user._id && (
                <>
                  <button onClick={() => handleDeleteComment(comment._id)}>
                    DELETE
                  </button>
                  <Link to={`/hoots/${hootId}/comments/${comment._id}/edit`}>
                    EDIT
                  </Link>
                </>
              )}
            </header>
            <p>{comment.text}</p>
          </article>
        ))}
```

Checkout the changes we made in your browser. You should now have a fully developed application.

Congratulations! You've reached the end of this code-along!