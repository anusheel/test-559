---
system_prompt: |
  You are an AI programming assistant, code generator and a documentation program.

  Each step of your response must strictly follow this markdown code block format :

  DESCRIPTION 

  ```LANG
  CODE
  ```
  where the tokens must be replaced such that:
  DESCRIPTION is a short but clear description of why you're doing that step. The description should include the fully qualified file name for the code block, including the file extension and the full directory path.
  LANG is the markup code block language for the code's language, and CODE is the code.

  Please note that the code should be fully functional. No placeholders.
  For any commands, add parameters to auto confirm defaults for any prompts on stdin. 
  Assume that the directory for the app has already been created and you are inside it, do not create a new directory.
  Use an env file for any variables like usernames, passwords, auth tokens or secrets.
model: gpt-4
temperature: 0
---

### Create your prompts below and Stub will generate the code and documentation


```stub
Create a Typescript and NextJS codebase that allows a user to authenticate using Github. 
Display all the repositories that the user has access to along with a link to the github repository.
Use Octokit package.

```

<details open id="511cf48ce08c7150c6415ac1cf501742">
<summary></summary>

First, we need to install the necessary packages. We will use `yarn` as our package manager. The packages we need are `next`, `react`, `react-dom` for the Next.js framework, `@types/node` and `@types/react` for TypeScript definitions, `typescript` for TypeScript language support, `dotenv` for environment variable management, `next-auth` for authentication, `@octokit/rest` for GitHub API client, and `isomorphic-unfetch` for making fetch requests on both client and server.

```bash
yarn add next react react-dom @types/node @types/react typescript dotenv next-auth @octokit/rest isomorphic-unfetch
```

Next, we need to create a `.env` file in the root directory of the project to store our GitHub OAuth application credentials. These credentials can be obtained by creating a new OAuth application in GitHub settings.

```bash
touch .env
```

Inside the `.env` file, add the following lines:

```env
GITHUB_ID=your_github_client_id
GITHUB_SECRET=your_github_client_secret
NEXTAUTH_URL=http://localhost:3000
```

Now, let's create a `next.config.js` file in the root directory of the project to load our environment variables.

```javascript
module.exports = {
  env: {
    GITHUB_ID: process.env.GITHUB_ID,
    GITHUB_SECRET: process.env.GITHUB_SECRET,
    NEXTAUTH_URL: process.env.NEXTAUTH_URL,
  },
};
```

Next, we need to set up NextAuth for GitHub authentication. Create a file named `[...nextauth].ts` inside the `pages/api/auth` directory.

```typescript
import NextAuth from 'next-auth'
import Providers from 'next-auth/providers'

export default NextAuth({
  providers: [
    Providers.GitHub({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
  ],
  callbacks: {
    async jwt(token, user) {
      if (user) {
        token.id = user.id
      }
      return token
    },
    async session(session, token) {
      session.user.id = token.id
      return session
    },
  },
})
```

Now, let's create a `pages/index.tsx` file to display the login button and the list of repositories.

```typescript
import { signIn, signOut, useSession } from 'next-auth/client'
import { Octokit } from '@octokit/rest'
import fetch from 'isomorphic-unfetch'

export default function Home() {
  const [session, loading] = useSession()

  if (loading) {
    return <div>Loading...</div>
  }

  if (!session) {
    return (
      <div>
        Not signed in <br />
        <button onClick={() => signIn('github')}>Sign in with GitHub</button>
      </div>
    )
  }

  return (
    <div>
      Signed in as {session.user.email} <br />
      <button onClick={() => signOut()}>Sign out</button>
      <RepoList session={session} />
    </div>
  )
}

function RepoList({ session }) {
  const [repos, setRepos] = React.useState([])

  React.useEffect(() => {
    const octokit = new Octokit({
      auth: session.accessToken,
    })

    octokit.repos
      .listForAuthenticatedUser()
      .then(({ data }) => {
        setRepos(data)
      })
      .catch((err) => console.error(err))
  }, [session])

  return (
    <ul>
      {repos.map((repo) => (
        <li key={repo.id}>
          <a href={repo.html_url} target="_blank" rel="noopener noreferrer">
            {repo.name}
          </a>
        </li>
      ))}
    </ul>
  )
}
```

Finally, add a `scripts` section to your `package.json` file to start the development server.

```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start"
}
```

Now, you can start the development server by running `yarn dev` in your terminal. Open `http://localhost:3000` in your browser to see the result.
</details>








