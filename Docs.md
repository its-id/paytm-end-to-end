## Understanding Architecture

1. When we click on "Pay now", the request goes to the HDFC bank server.

2. The bank server responds back with a token to the backend.

3. The Backend provides a url for the frontend to redirect to the bank's payment page.

4. The bank's payment page is shown to the user.

5. After user pays the amount, the bank redirects the user back to the frontend with a response.

## Understanding Authentication

1. Go to the `apps/user-app/api/auth/[...nextauth]/route.ts` file.

2. Any request to the `/api/auth` endpoint is handled by the `next-auth` package.

3. The `next-auth` package is used to authenticate the user. Let's go deep inside the `authOptions` being imported inside this file.

4. The `authOptions` is our main configuration file for authenticating the user. It is being exported from `app/lib/auth.ts`

   - `providers: [..]` : This is where we define the providers for authentication. We are using phone no. and password for authentication.

   - `name: 'Credentials` : This is the name of the provider. Will be shown in the login page button.

   - `credentials: {..}` : This is where we define the fields required for authentication.

   - `authorize: async (credentials) => {..}` : This is where we define the main logic for authenticating the user.

     - `return {...}` : We need to return an object with the user data if the user is authenticated else return null.

   - `callbacks: {..}` : These are the callbacks that are called after the user is authenticated.
     - `async session(session, user) {..}` : After the user is authenticated, We can add the user data to the session here.
     - `session.user.id = token.sub` : This is how we add the user id to the session.
     - `return session`: We need to return the session object. It will be available in the `req` object in the API routes.

## Understanding Password Hashing

1. When a user signs up, we can see the password being hashed in the database.

2. This is done using a package called [`bcrypt`](https://www.npmjs.com/package/bcrypt).

3. If we continue looking at the code inside `async authorize(credentials)` , we can see the password being hashed using the `bcrypt` package.

## Understanding the Next Steps

1. We have created the basic user and merchant ui with authentications.

2. Today's goal is to create

   - a dummy bank server.
   - a pure backend server which will work as a webhook handler that will handle the bank's response and payment requests.

3. Go to `bank_webhook_handler` folder to see next steps.
