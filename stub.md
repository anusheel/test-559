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

<details open id="a2d0f58230841b9a1ca80faebe0060a4">
<summary></summary>

To create a Next.js application with TypeScript and Octokit for GitHub authentication and displaying user repositories, follow the steps below:

First, install the necessary packages:

```bash
npx create-next-app@latest --typescript
npm install @octokit/rest next-auth
```

Next, create a file named `.../pages/api/auth/[...nextauth].ts` and add the following code:

```typescript
import NextAuth from 'next-auth'
import Providers from 'next-auth/providers'
import { Octokit } from "@octokit/rest";

export default NextAuth({
  providers: [
    Providers.GitHub({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
      scope: 'repo'
    })
  ],
  callbacks: {
    async jwt(token, user) {
      if (user) {
        const octokit = new Octokit({
          auth: user.accessToken,
        });

        const { data } = await octokit.repos.listForAuthenticatedUser();
        token.userRepos = data;
      }
      return token
    },
    async session(session, token) {
      session.user.repos = token.userRepos;
      return session
    }
  }
})
```

In this file, we are using the GitHub provider for NextAuth.js and setting the scope to 'repo' to get access to the user's repositories. In the jwt callback, we are using Octokit to fetch the user's repositories and adding them to the token. In the session callback, we are adding the repositories to the session.

Next, create a `.env.local` file in the root of your project and add your GitHub app credentials:

```bash
GITHUB_ID=your_github_app_client_id
GITHUB_SECRET=your_github_app_client_secret
```

Finally, create a file named `.../pages/index.tsx` and add the following code:

```typescript
import { signIn, signOut, useSession } from 'next-auth/client'

export default function Home() {
  const [session, loading] = useSession()

  if (loading) {
    return <div>Loading...</div>
  }

  if (!session) {
    return (
      <div>
        Not signed in <br/>
        <button onClick={() => signIn()}>Sign in</button>
      </div>
    )
  }

  return (
    <div>
      Signed in as {session.user.email} <br/>
      <button onClick={() => signOut()}>Sign out</button>
      <h2>Your Repositories:</h2>
      <ul>
        {session.user.repos.map((repo: any) => (
          <li key={repo.id}>
            <a href={repo.html_url} target="_blank">{repo.name}</a>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

In this file, we are using the useSession hook from NextAuth.js to get the current user session. If the user is not signed in, we display a sign in button. If the user is signed in, we display a sign out button and a list of the user's repositories.

Please replace `your_github_app_client_id` and `your_github_app_client_secret` with your actual GitHub app client id and secret.
</details>


