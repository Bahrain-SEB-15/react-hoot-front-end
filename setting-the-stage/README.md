# ![React Hoot - Setting the Stage](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to conceptualize the required features and high level architecture of this application.

## What we're building - Hoot: Blogging for night owls

In this module, we are going to create a frontend blogging application using React and JWT Authentication. Pairing this frontend app with an Express API backend and MongoDB database will yield a complete MERN Stack application.

In our app, through JWT Authentication, users will be able to sign up, sign in, and logout.

Users with an account will be able to create, read, update and delete blog posts. For branding purposes, we'll refer to these blog posts as 'hoots'. Additionally, logged in users will be able to create and read comments associated with a specific hoot post.

In building this application, you'll get hands on experience with several patterns fundamental to React, including client-side routing with React Router DOM, dynamically rendering content based on permissions, and implementing reusable components.

Take a look at the screenshots below for a better sense of the application:

![Screenshot tktk Hunter]()

> 💡 Note, the screenshots above depict the application after completing all provided level ups.

##  User stories

Below are the user stories we will implement within Hoot:

- As a guest, I should be able to create an account.
- As a guest with an account, I should be able to log in to my account.
- AAU, I should be able to create a hoot post.
- AAU, I should be able to see a list of all hoots on a 'List' page.
- AAU, clicking on a hoot in the 'List' page should navigate me to a 'Details' page where I can view information about a single hoot post along with its associated comments.
- AAU, I should be able to add a comment on a hoot 'Details' page.
- As the author of a hoot, I should see a link to 'Edit' a hoot on the 'Details' page. Clicking on the link should direct me to an 'Edit' page where I can modify the hoot. Upon submitting the update, I should be redirected back the the 'Details' page.
- As the author of a hoot, I should see a button to 'Delete' a hoot on the 'Details' page. Clicking on the button should delete the hoot, and redirect me back to the 'List' page.

You might notice that the above user stories give us a good idea of what CRUD functions a user might want to perform in our applications.  

## Component hierarchy diagram

After reviewing the user stories, our next step is to map out the structure of our React app. For this, we'll utilize a **Component Hierarchy Diagram**. This visual tool will act as an outline for the tree structure of components in our client-side application. 

Below is an example of the component hierarchy diagram we will be using for Hoot:

![Component hierarchy diagram tktk Hunter]()