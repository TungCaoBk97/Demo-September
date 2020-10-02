# Build Spring Boot app using Keycloak, custom authentication flow in Keycloak
Build a simple library management application and authentication user using Keycloak. 
- Application has 2 roles: Member and Librarian
- 3 endpoint with different from accessible: `/index`, `/books` and `/manager`.
- Login is performed in Keycloak with some flow, using `Username/Password form` or using verify `Email link` or answer the `Secret question`
- Fake smtp server using `MailHog`

# Installation
## Keycloak
- Download: https://www.keycloak.org/downloads
- Unzip the downloaded zip file (i.e unzip keycloak-11.0.2)
- Start server (Wildfly): `bin/standalone.sh`, Keycloak server will start at `localhost:8080`

## MailHog
- Install in Ubuntu: 
```bash
    sudo apt-get -y install golang-go
    go get github.com/mailhog/MailHog
```
- Then, start MailHog by running `~go/bin/MailHog` in the command line.

# Configuration
## **Keycloak**

## Creating the realm

Open [Keycloak Admin Console](http://localhost:8080/auth/admin/). Login with username and password for admin account.

Create a new realm called `public-library` (find the `add realm` button in the drop-down in the top-left corner). 

## Create a client

Now create a client for the Spring app by clicking on `clients` then `create`.

Fill in the following values:

* Client ID: `spring-boot-app`
* Click `Save`

On the next form fill in the following values:

* Valid Redirect URIs: `http://localhost:8000/*`
* Web Origins: `http://localhost:8000`

## Configuring SMTP server

From the drop-down in the top-left corner select `Master`. Go to `Users`, click on `View all users` and select the `admin` user. Set the `Email` field to `admin@localhost` (only to test email delivery).

Now switch back to the `public-library` realm, then click on `Realm Settings` then `Email`. 

Fill in the following values:

* Host: `localhost`
* Port: 1025
* From: `keycloak@localhost`

Click `Save` and `Test connection`. Open your http://localhost:8025 and check that you have received an email from Keycloak.

## Enable user registration

Let's enable user self-registration and at the same time require users to verify their email address.

Open the [Keycloak Admin Console](http://localhost:8080/auth/admin/). 

Click on `Realm settings` then `Login`.

Fill in the following values:

* User registration: `ON`
* Verify email: `ON`

## Create Roles

Click on `Roles` and `Add Role`. Set the Role Name to `Member` and click `Save`. Add another role name `Librarian`.

Now click on `Users` and create a new user you want to login with. Click on `Role Mappings`. 
Select `Member` or `Librarian` from Available roles and click `Add selected`. (i.e `user1` has `Member` role, `user2` has `Librarian` role).


# **Spring project**

Create a Spring project name spring-boot-app (see the folder above). Configure application for using Keycloak to authenticate user in `application.properties`

    keycloak.realm=public-library
    keycloak.resource=spring-boot-app
    keycloak.credentials.secret=${CLIENT_SECRET}
    keycloak.auth-server-url=http://localhost:8080/auth
    #keycloak.ssl-required=external
    #keycloak.public-client=true
    keycloak.principal-attribute=preferred_username
    server.port=8000

Run Spring project With Gradle

    ./gradlew bootRun

With Maven

    ./mvnw spring-boot:run

Application will run in [localhost://8000](http://localhost:8000).

Example Home page

![Home page](/images/home-page.png)

Login page

![Login page](/images/login-page.png)

`/books` page

![Books page](/images/books-page.png)

`/manager` page

![Manager page](/images/manage-page.png)

Forbidden page

![Forbidden page](/images/forbidden-page.png)


# Custom authenticators in Keycloak
## First, deploy `magic-link` custom execution
This option is default by Keycloak, but this is removed from keycloak version 3.0+. So we must to build it and deploy to Keycloak (see folder magic-link). We are easy to find this source in Keycloak github official.

### Usage

    mvn clean install wildfly:deploy

Now magic-link execution is available 

## Config realm authentication flow
Includes a custom authenticator that enables users to login through email

Open the [Keycloak Admin Console](http://localhost:8080/auth/admin/).  Click on `Authentication`. 

Click on `Copy` to create a new flow based on the `browser` flow. Use the name `My Email link Flow`.

Click on `Actions` and `Delete` for `Username Password Form` and `OTP Form`.

Click on `Actions` next to `My Email link Flow Forms`. Then click on `Add execution`.
Select `Magic Link` from the list. Once it's saved select `Required` for the `Magic Link`.

Now to use this new flow when users login select `Bindings` and select `My Email link Flow`.

Open the [spring-boot-app](http://localhost:8000) and click in one protected tab. For the email enter your email address and click `Log In`. Open your email in [Mailhog](http://localhost:8025) and you should have a mail with a link which will authenticate you and bring you to the `spring-boot-app`.

Example Login with email

![Login with email](/images/email-login.png)

Check email in [Mailhog](http://localhost:8025) and click this link. It will move you to the application

![Check email](/images/view-email.png)


## Another custome authenticator flow
### With Secret Question
See folder authenticators (this source code is easy to find in Keycloak github). 

Deploy to Wildfly server

    mvn clean install wildfly:deploy

Copy the `secret-question.ftl` and `secret-question-config.ftl` files to the `themes/base/login` server directory to display the question box. 

Do the same action with `magic-link`, create new authentication flow name `My secret question`, add execution `Secret question` and Binhding it.

Example Secret question

![Secret question](/images/secret-question.png)



