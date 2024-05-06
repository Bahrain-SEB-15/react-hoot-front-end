# ![React - Hoot Front-End - Setup](./assets/hero.png)

## Setup

Open your Terminal application and navigate to your `~/code/ga/lectures` directory:

```bash
cd ~/code/ga/lectures
```

## Cloning the Auth boilerplate

This lecture uses the [React JWT Auth Template](https://git.generalassemb.ly/modular-curriculum-all-courses/react-jwt-auth-template.git) as starter code. The template includes code to authenticate users in React using JWT tokens generated from an existing Express backend API.

Navigate to the `React JWT Auth Template` and clone the repository to your machine:

```bash
git clone https://git.generalassemb.ly/modular-curriculum-all-courses/react-jwt-auth-template.git
```

Once we have the repository on our machines, we can change the name of the directory to `'react-hoot-front-end'`:

```bash
mv react-jwt-auth-template react-hoot-front-end
```

Next, `cd` into your renamed directory:

```bash
cd react-hoot-front-end
```

Finally, remove the existing `.git` information from this template:

```bash
rm -rf .git
```

> Removing the `.git` info is important as this is just a starter template provided by GA. You do not need the existing git history for this project.

## GitHub setup

To add this project to GitHub, initialize a new Git repository:

```bash
git init
git add .
git commit -m "init commit"
```

Make a new repository on [GitHub](https://github.com/) named `react-hoot-front-end`. 

Link your local project to your remote GitHub repo:

```bash
git remote add origin https://github.com/<github-username>/react-hoot-front-end.git
git push origin main
```

> 🚨 Do not copy the above command. It will not work. Your GitHub username will replace `<github-username>` (including the `<` and `>`) in the URL above.

Open the project's folder in your code editor:

```bash
code .
```

## Install dependencies

Next, you will want to install all of the packages listed in `package.json`

```bash
npm i
```

## Create a `.env`

Run the following command in your terminal:

```bash
touch .env
```

Lastly, we want to include a `VITE_EXPRESS_BACKEND_URL`.

Add the following secret key to your `.env`:

```text
VITE_EXPRESS_BACKEND_URL="http://localhost:3000"
```

## Update the `.gitignore`

Add `package-lock.json` and `.env` to the `.gitignore` file.  

```text
node_modules
package-lock.json
.env
```

## Start your application

Start the application with the following command:

```bash
npm run dev
```

## Running the Express backend

Before diving into our React app development, you'll need to ensure that the Express backend server is operational. This backend will handle requests from your React app. You will be using the back-end server you created in the [`Express API - Hoot Back-End`](https://git.generalassemb.ly/modular-curriculum-all-courses/express-api-hoot-back-end) lesson as the API for this lesson.

Follow these steps to set up the server:

Open your Terminal application and navigate to your **`~/code/ga/lectures/express-api-hoot-back-end`** directory:

```bash
cd ~/code/ga/lectures/express-api-hoot-back-end
```

Once there, run your server with `nodemon`:

```bash
nodemon server.js
```

> Note: If your `express-api-hoot-back-end` is incomplete, you can obtain a fully implemented version from the [solution code repo](https://git.generalassemb.ly/modular-curriculum-all-courses/express-api-hoot-back-end-solution). Remember to install all necessary dependencies with `npm i` and establish a connection to your MongoDB Atlas by adding a connection string in a `.env` file.

### Create your server `.env`

To configure your server using the provided starter code, you'll need to set up a `.env` file that includes both the `MONGODB_URI` and `JWT_SECRET`:

1. **Establish a MongoDB Connection:**
   - Sign up or log into MongoDB Atlas and create a new database cluster. 
   - Generate a connection string for your MongoDB Atlas database, which will be used as your `MONGODB_URI`.

2. **Create JWT_SECRET:**
   - The `JWT_SECRET` is a secret key used for signing your JWT tokens. Choose a secure and random string.

3. **Set Up Your `.env` File:**
   - In the root directory of your server application, create a `.env` file.
   - Add your MongoDB connection string and JWT secret to this file as follows:

```plaintext
MONGODB_URI=your_mongodb_atlas_connection_string_here
JWT_SECRET=your_secure_random_string_here
```

Start the server and you are ready to start on the React front-end!

```bash
nodemon start
```

Happy Coding!