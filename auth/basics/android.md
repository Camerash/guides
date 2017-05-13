---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in Android
---

[[toc]]

## Overview
Skygear support user authentication with email or username. The following
shows how it works using Skygear SDK.

The global `Container` named `skygear` will be used throughout the examples:
```java
Container skygear = Container.defaultContainer(this); // Global Container
```

Each user, when logged in, will be given a generated String called `Access Token`, which is like a identification card for the user. It is used by the Skygear server to identify who are you.

Example of usage of `Access Token` is the method `whoami`, which will send the current `Access Token` to the server to return a `User` object.

## Signing up / Logging in / Logging out

### Signing up

A user can sign up using:
1. Username
2. Email

The following shows how to sign up a user with username and password.

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.signupWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Signing up with email is done with much the same way:

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.signupWithEmail(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```
However, if user try to sign up with an exsisting username / email, the error `ERROR.CODE.DUPLICATED` will be returned in the `onAuthFail` method.

A typical account checking example is shown below:
```java
skygear.signupWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        if(error.getCode().equals(Error.Code.DUPLICATED)){
            Toast.makeText(getApplicationContext(), "User already exist!", Toast.LENGTH_SHORT).show();
            //Duplicate user handling...
        }
    }
});
```

### Anonymous user

Skygear supports users to sign up anonymously. They act like normal users and can read and update records.

To create an anonymous user, call `skygear.signupAnonymously()` as shown below:

```java
skygear.signupAnonymously(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Each anonymous user has a unique ID that behaves exactly the same way as normal. As expected, an anonymous user does not have username, email nor password.

Because of this behaviour, once the token of an anonymous user is lost, the account will not be able to access again. The records, however, will still persist.

### Logging in

Logging in behave much the same way as signing up a user.

The following shows how to log in a user with username and password.

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.loginWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Users can log in with their emails if they have registered:

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.loginWithUsername(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

### Logging out
To logout from the current user, simply call `skygear.logout()` as shown below:

```java
skygear.logout(new LogoutResponseHandler() {
   @Override
   public void onLogoutSuccess() {
       Log.i("Skygear Logout", "Successfully logged out");
   }

   @Override
   public void onLogoutFail(Error error) {
        Log.i("Skygear Logout", "onLogoutFail: Fail with reason:" + error.getMessage());
   }
  
});
```

## Getting the current User

After sign up / log in, the user session can be obtained by:

```java
User currentUser = skygear.getCurrentUser(); // Return null if not logged in

Log.i("Skygear User", "User access token: " + currentUser.accessToken);
Log.i("Skygear User", "User ID: " + currentUser.userId);
Log.i("Skygear User", "Username: " + currentUser.username);
```

Please be reminded that the `currentUser` object persist locally, and the
information (e.g. roles, emails, etc) might not sync with the server if it was
changed remotely.

To get the latest information of the current user, you can call `whoami()`.

```java
skygear.whoami(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear User", "I am " + user.getUsername());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear User", "onAuthFail: Fail with reason:" + error.getMessage());
    }
});
```

## Changing username/email/password

### Change the username / email of a user

To change a user's username and email, you can use the `skygear.saveUser()` method by providing the user ID and the new username and/or the new email.

Each user can only edit his/her information
While only users with the admin role can change information of other users.

To change the username/email of the current user:
```java
User currentUser = skygear.getCurrentUser();

User newUser = new User(currentUser.getId(), 
                        currentUser.getAccessToken(), 
                        "your-new-username",
                        "your-new-email@hello.com"); 
                        
skygear.saveUser(newUser, new UserSaveResponseHandler() {
    @Override
    public void onSaveSuccess(User user) {
        Log.i("Skygear User", "Update user successful");
    }

    @Override
    public void onSaveFail(Error error) {
        Log.i("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
    }
});
```

If you only want to change one of the user field, just replace the new username/email with `currentUser.getUsername()`/`currentUser.getEmail()`

### Change the password of a user
Coming Soon

## What's next from here?

You may want to learn more about:
- [Social Login using Skygear (Coming soon)](https://docs.skygear.io/guides/auth/social-login/android/)
- [Setting User Profile (Coming soon)](https://docs.skygear.io/guides/auth/user-profile/android/)
