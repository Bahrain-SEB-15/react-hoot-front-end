# ![React Hoot - Setup](./assets/hero.png)

## Setup

Open your Terminal application and navigate to your `~/code/ga/lectures` directory:

```bash
cd ~/code/ga/lectures
```

## Cloning the Auth boilerplate

This lecture uses the [React JWT Auth Template](https://git.generalassemb.ly/modular-curriculum-all-courses/react-jwt-auth-template.git) as starter code. The template includes code to authenticate users in React using JWT tokens generated from their own Express backend API.

Navigate to the [React JWT Auth Template](https://git.generalassemb.ly/modular-curriculum-all-courses/men-stack-session-auth-template) and clone the repository to your machine:

```bash
git clone git@git.generalassemb.ly:modular-curriculum-all-courses/react-jwt-auth-template.git
```

Once we have the repository on our machines, we can change the name of the directory to `'react-hoot'`:

```bash
mv react-jwt-auth-template react-hoot
```

Next, `cd` into your renamed directory:

```bash
cd react-hoot
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

Make a new repository on [GitHub](https://github.com/) named `react-hoot`. 

Link your local project to your remote GitHub repo:

```bash
git remote add origin https://github.com/<github-username>/react-hoot.git
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

## Create your .gitignore

Run the following command in your terminal:

```bash
touch .gitignore
```

Once these files are created, add `.env`, `package-lock.json`, and `node_modules` to your `.gitignore` file.  Doing so will prevent those files and directories from being tracked and we can be confident that any data we add there will not be pushed up to GitHub.

```text
.env
node_modules
package-lock.json
```

## Create your .env

Run the following command in your terminal:

```bash
touch .env
```

Lastly, we want to create a `VITE_EXPRESS_BACKEND_URL`.

Add a `.env` file to your application and add the following secret keys to your application:

```text
VITE_EXPRESS_BACKEND_URL="http://localhost:3000"
```

Start the application with the following command:

```bash
npm run dev
```

Happy Coding! 