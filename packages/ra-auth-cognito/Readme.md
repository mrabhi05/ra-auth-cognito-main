# ra-auth-cognito

An auth provider for [react-admin](https://github.com/marmelab/react-admin) which handles authentication using AWS [Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html).

This package provides:

-   The `CognitoAuthProvider` function to get the auth provider
-   A `useCognitoLogin` hook to allow building a custom `Login` page. It handles initial login with a temporary password
-   A custom `Login` component that handle initial login with a temporary password

## Supported Cognito Features

- Username/password authentication
- OAuth authentication with Implicit code grant

In all cases, users must be added to the user pool with their email set before they may sign-in in react-admin.

## Installation

```sh
yarn add ra-auth-cognito
# or
npm install --save ra-auth-cognito
```

## Usage With Username/Password Sign-in

When not using the AWS hosted UI, users you create in AWS will receive an email with a temporary password. The first time they log in the application with this temporary password, they will have to enter the password they want to use. To handle this use case, `ra-auth-cognito` provides a custom `<Login>` component that you can pass to you `<Admin>` through the `loginPage` prop:

```jsx
// in src/App.tsx
import React from 'react';
import { Admin, Resource } from 'react-admin';
import { CognitoAuthProvider, Login } from 'ra-auth-cognito';
import { CognitoUserPool } from 'amazon-cognito-identity-js';
import dataProvider from './dataProvider';
import posts from './posts';

const userPool = new CognitoUserPool({
    UserPoolId: 'COGNITO_USERPOOL_ID',
    ClientId: 'COGNITO_APP_CLIENT_ID',
});

const authProvider = CognitoAuthProvider(userPool);

const App = () => {
  return (
       <Admin
           authProvider={authProvider}
           dataProvider={dataProvider}
           title="Example Admin"
           loginPage={Login}
        >
            <Resource name="posts" {...posts} />
      </Admin>
   );
};
export default App;
```

If you need to customize this login page, please refer to the [`<LoginForm>` component](#loginform) and [`useCognitoLogin` hook](#usecognitologin) documentation.

## Usage With AWS Hosted UI (OAuth)

```jsx
// in src/App.tsx
import React from 'react';
import { Admin, Resource } from 'react-admin';
import { CognitoAuthProvider } from 'ra-auth-cognito';
import { CognitoUserPool } from 'amazon-cognito-identity-js';
import dataProvider from './dataProvider';
import posts from './posts';

const authProvider = CognitoAuthProvider({
    mode: 'oauth',
    clientId: 'COGNITO_APP_CLIENT_ID',
    userPoolId: 'COGNITO_USERPOOL_ID',
    hostedUIUrl:  'YOUR AWS HOSTED UI URL',
});

const App = () => {
  return (
       <Admin
           authProvider={authProvider}
           dataProvider={dataProvider}
           title="Example Admin"
           loginPage={false} // We don't need the login page in this case
        >
            <Resource name="posts" {...posts} />
      </Admin>
   );
};
export default App;
```

## Handling User Identities

To support react-admin [identity feature](https://marmelab.com/react-admin/AuthProviderWriting.html#getidentity), you may add the `name` and `picture` [attributes](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html) to the users registered in your user pool.

## Handling Permissions

This `authProvider.getPermissions` method returns an array of [the groups assigned to the user](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-user-groups.html?icmpid=docs_cognito_console_help_panel).

## `<LoginForm>`

A component that renders a login form. It handles first login with temporary passwords. Use it if you just want to customize the login page design:

```tsx
import { Box, Card, CardContent, CardMedia, CssBaseline } from '@mui/material';
import { LoginForm } from 'ra-auth-cognito';

export const MyLoginPage = () => {
    return (
		<>
			<CssBaseline />
			<Box>
                <Card>
                    <CardMedia
                        sx={{ height: 140 }}
                        image="/login_background.jpg"
                    />
                    <CardContent>
                        <LoginForm redirectTo="/" />
                    </CardContent>
				</Card>
			</Box>
		</>
	);
}
```

## `useCognitoLogin`

This hook will handle the login process, detecting whether users must provide their new password when they logged in with a temporary one. This is useful when you want complete control on your login UI:

```tsx
import { Box, Card, CardContent, CardMedia, CssBaseline } from '@mui/material';
import { useCognitoLogin } from 'ra-auth-cognito';
import { LoginForm } from './LoginForm';
import { PasswordSetupForm } from './PasswordSetupForm';

export const MyLoginPage = () => {
    const [submit, { isLoading, requireNewPassword }] = useCognitoLogin({
        redirectTo: '/'
    });

    return (
		<>
			<CssBaseline />
			<Box>
                <Card>
                    <CardMedia
                        sx={{ height: 140 }}
                        image="/login_background.jpg"
                    />
                    <CardContent>
                        {requireNewPassword
                            ? <PasswordSetupForm onSubmit={submit} />
                            : <LoginForm onSubmit={submit} />
                        }
                    </CardContent>
				</Card>
			</Box>
		</>
	);
}
```

## Demo

You can find a working demo, along with the source code, in this project's repository: https://github.com/marmelab/ra-auth-cognito

## License

This auth provider is licensed under the MIT License and sponsored by [marmelab](https://marmelab.com).
