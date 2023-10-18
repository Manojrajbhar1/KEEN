# Implementing Keycloak SSO for both Redmine and Superset

## **Integrate Redmine with keycloak**

<aside>
💡

## **Integrate Redmine with keycloak**

- **Redmine**: Redmine serves as the issue management platform, enabling efficient tracking and handling of transportation-related issue-tickets. It centralizes issue reporting, assignment, and resolution.
- **Superset:** Superset functions as a powerful data visualization tool, providing interactive and insightful dashboards and charts. It aids in transforming raw data into comprehensible visual representations for informed decision-making.
- **Keycloak:** Keycloak plays a pivotal role in identity and access management. It facilitates a single sign-on (SSO) solution, ensuring that users can access both Redmine and Superset with a single set of credentials, enhancing user convenience and security.
- **PostgreSQL:**PostgreSQL serves as the underlying database management system, storing and organizing the data used by Redmine and Superset. It provides a reliable foundation for data storage and retrieval.

*By seamlessly integrating Redmine, Superset, Keycloak, and PostgreSQL, the toolset offers an efficient and comprehensive solution for managing transportation-related issues, visualizing data, streamlining user access, and maintaining a robust database environment.*

</aside>

# **Why did we set up the keycloak?**

<aside>
💡

## **Problem:**

In the absence of a Single Sign-On (SSO) solution, users accessing the Redmine issue management tool and the Apache Superset data visualization tool were required to log in separately. This dual authentication process was time-consuming and potentially burdensome, especially for users who frequently accessed both tools.

## **Solution:**

Keycloak: Simplifying User Access with Single Sign-On (SSO)

To alleviate the inconvenience of separate logins and streamline the user experience, a Single Sign-On solution called Keycloak was introduced. Keycloak acts as an intermediary between Redmine and Superset, offering users a unified authentication process.

## **Benefits:**

- Unified User Authentication: With Keycloak, users need to authenticate only once. Once logged in, they gain seamless access to both the Redmine and Superset platforms, eliminating the need for redundant logins.
- Enhanced User Experience: The convenience of SSO significantly enhances user experience, reducing friction in the authentication process and improving user satisfaction.
- Efficiency and Productivity: Users can seamlessly transition between Redmine and Superset without interruption. This efficiency boost leads to enhanced productivity and smoother workflow.

</aside>

<aside>
💡

## **Environment :-**

Redmine version 4.0.5.stable

Keycloak 22.0.3

PostgreSQL 16.0

Superset

## **OS Details :-**

Ubuntu 20.04

</aside>

<aside>
💡 **If you have not podman installed , then follow the below command**

</aside>

```bash
**echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L"https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key" | sudo apt-key add -
sudo apt update
sudo apt install podman**
```

<aside>
💡 A Dockerfile is a script used to create a Docker image. Docker containers are instances of Docker images, and they run in isolated environments on a host system. We are going to create a Redmine Dockerfile, build it, and then run the containers

</aside>

<aside>
💡 First we have to create Dockerfile for redmine
**Touch Dockerfile:** is generally used for creating a file
**Vim Dockerfile:** vim is generally used for editing inside the file
**Cat Dockerfile:** cat is used for seeing the content inside the file

</aside>

```bash
**manoj@crunchy-postgres:~$ cat Dockerfile
# syntax=docker/dockerfile:1
FROM docker.io/library/redmine:4.0.5
# Switching to root to install the required packages
WORKDIR /usr/src/redmine/plugins
RUN git clone https://github.com/onozaty/redmine-view-customize.git view_customize
RUN git clone https://github.com/devopskube/redmine_openid_connect.git
WORKDIR /usr/src/redmine/
RUN bundle install --without development test**
```

---

<aside>
💡 Now build the images here with below command

</aside>

```bash
**manoj@crunchy-postgres:~$ podman build -t redmine:manoj .
STEP 1/6: FROM docker.io/library/redmine:4.0.5
STEP 2/6: WORKDIR /usr/src/redmine/plugins
--> Using cache 0f62ab8008e7798deb7dfe5934edfde686ea2b51fd2eccfa5a7b4301d49f5efe
--> 0f62ab8008e
STEP 3/6: RUN git clone https://github.com/onozaty/redmine-view-customize.git view_customize
--> Using cache 73c2576456d2def7ea16bd0b0352667edc7564f99301a0db97caaa6354d1bea9
--> 73c2576456d
STEP 4/6: RUN git clone https://github.com/devopskube/redmine_openid_connect.git
--> Using cache 43bbfa8dc026e93b96477ba51d2f963bbda7a9526e29e62184fd3d0cfc32f47c
--> 43bbfa8dc02
STEP 5/6: WORKDIR /usr/src/redmine/
--> Using cache 018c6ed3a78b3a1446a9c682276be6d5b83d0db99e4e9fe3333c02a2883ea807
--> 018c6ed3a78
STEP 6/6: RUN bundle install --without development test
--> Using cache 36f8102faa89c459c9621e2a8dbde6b7337426684a0fcdb855fde0cc54cff0f5
COMMIT redmine:manoj
--> 36f8102faa8
Successfully tagged localhost/redmine:manoj
Successfully tagged localhost/redmine_custom:1.0
36f8102faa89c459c9621e2a8dbde6b7337426684a0fcdb855fde0cc54cff0f5**
```

---

<aside>
💡 Check the build images are present or not

</aside>

```bash
**manoj@crunchy-postgres:~$ podman images
REPOSITORY TAG IMAGE ID CREATED SIZE
localhost/superset_apache mj 37735fa096a4 4 days ago 1.33 GB
localhost/redmine_custom 1.0 36f8102faa89 5 days ago 553 MB
localhost/redmine manoj 36f8102faa89 5 days ago 553 MB**
```

---

<aside>
💡 I have run a Redmine container, built from the created image, and connected it to a PostgreSQL database.

</aside>

<aside>
💡 **Now we are creating a file of keycloak.sh**

</aside>

```bash
**manoj@crunchy-postgres:~$ touch keycloak.sh**
```

<aside>
💡 **Copy the below bash script inside the keycloak.sh file**

</aside>

<aside>
💡 **For checking the inside content we can use the command of cat and file name**

</aside>

```bash
**manoj@crunchy-postgres:~$ cat keycloak.sh**
```

```bash
**#!/bin/bash
# Create a folder for PostgreSQL data
echo "Creating folder for PostgreSQL data"
mkdir -p /home/manoj/data/redmine/postgres
# Change ownership of the data folder
echo "Changing ownership of the data folder"
podman unshare chown 999:999 /home/manoj/data/redmine/postgres
# Create a Podman pod for Redmine, Keycloak, and Superset
echo "Creating the Redmine pod"
podman pod create --name redmine \
--publish 3000:3000 \
--publish 5432:5432 \
# Set up PostgreSQL container
echo "Running the PostgreSQL container"
podman run -dt \
--pod redmine \
--name redmine-postgres \
-e POSTGRES_DB=redmine \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=Password \
-e POSTGRES_HOST_AUTH_METHOD=trust \
-e PGDATA=/var/lib/postgresql/data/pgdata \
-v /home/manoj/data/redmine/postgres:/var/lib/postgresql/data \
docker.io/postgres:latest
# Set up Redmine container
echo "Running the Redmine container"
podman run -dt \
--pod redmine \
--name redmine-app \
-e REDMINE_DB_POSTGRES=127.0.0.1 \
-e REDMINE_DB_PORT=5432 \
-e REDMINE_DB_DATABASE=redmine \
-e REDMINE_DB_USERNAME=postgres \
-e REDMINE_DB_PASSWORD=password \
redmine:manoj**
```

---

**Give execute permission to above script by +x filename**

```bash
**chmod +x keycloak.sh**
```

**After giving execute permission Run below command for execute the script**

```bash
**manoj@crunchy-postgres:~$ podman exec -it redmine-app bash
root@b17c47e5f17a:/usr/src/redmine# bundle install
The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32`.
Using rake 13.0.1
Using concurrent-ruby 1.1.5
Using i18n 0.7.0
Using sqlite3 1.3.13
Bundle complete! 29 Gemfile dependencies, 61 gems now installed.
Gems in the groups development and test were not installed.
Bundled gems are installed into `/usr/local/bundle`
root@b17c47e5f17a:/usr/src/redmine#
root@b17c47e5f17a:/usr/src/redmine# cd plugins/
root@b17c47e5f17a:/usr/src/redmine/plugins# ls
README redmine_openid_connect view_customize
root@b17c47e5f17a:/usr/src/redmine/plugins# bundle install
The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32`.
Using rake 13.0.1
Using concurrent-ruby 1.1.5
Using i18n 0.7.0
Using minitest 5.13.0
root@b17c47e5f17a:/usr/src/redmine/plugins# cd redmine_openid_connect/**
```

```bash
**root@b17c47e5f17a:/usr/src/redmine/plugins/redmine_openid_connect# bundle install
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Using bundler 1.17.2
Using multi_xml 0.6.0
Using httparty 0.14.0
Bundle complete! 1 Gemfile dependency, 3 gems now installed.
Gems in the groups development and test were not installed.
Bundled gems are installed into `/usr/local/bundle`
root@b17c47e5f17a:/usr/src/redmine/plugins/redmine_openid_connect#
root@b17c47e5f17a:/usr/src/redmine/plugins/view_customize# bundle install
Resolving dependencies...
Using i18n 0.7.0
Using minitest 5.13.0
Using activerecord-compatible_legacy_migration 0.1.2
Bundle complete! 1 Gemfile dependency, 11 gems now installed.
Gems in the groups development and test were not installed.
Bundled gems are installed into `/usr/local/bundle`**
```

```bash
**root@b17c47e5f17a:/usr/src/redmine/plugins/view_customize#
#run this command in last then restart
root@b17c47e5f17a:/usr/src/redmine/plugins/view_customize#**
```

## **After restart the container you have to run below command**

```bash
**manoj@crunchy-postgres:~$** **podman restart redmine-app**
```

```bash
**manoj@crunchy-postgres:~$ podman exec -it redmine-app bash
root@b17c47e5f17a:/usr/src/redmine# bundle exec rake redmine:plugins:migrate RAILS_ENV=production
manoj@crunchy-postgres:~$ podman restart redmine-app
b17c47e5f17a94bdbaab3ced7dea0c9c5c3368893c5463957af75e94a7b8b8c7**
```

---

**Creating Keycloak container**

```bash
**manoj@crunchy-postgres:~$ podman run -d --name keycloak -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:22.0.3 start-dev**
```

---

## **Create a REALM role as needed !!**

<aside>
💡

**6 - Keycloak Configuration for Redmine and Superset -**

**Note for SS0 to work properly create the same client for redmine and superset.**

**Create a client scope named openid in Clinet_Scopes -**

**Add the following mappers -**

**username -- of type user property -- token claim name should be - user_name**

**member_of -- of type user realm role -- token claim name should be member_of**

**roles --- of type user realm role -- token claim name should be roles**

**add all these mappers to id_token , userinfo and access_token.**

**5 - Open keycloak Admin ui and create a REALM role Admin for Admin user in superset and redmine . User for Normal user in redmine .**

**User Role Supported By Redmine -**

**Admin**

**User**

**User Role Supported By Superset -**

**Admin**

**Alpha**

**Gamma**

**Public**

**granter**

**sql_lab**

</aside>

**Hit on browsers http://192.168.122.199:8080**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image13.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image13.png)

- **Go to left side after sign in**

**Create realm >Keenable**

**And client name>keenable**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image43.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image43.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image49.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image49.png)

**Go to client section and create client name keenable**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image1.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image1.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image51.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image51.png)

<aside>
💡 **Study from here first ,then go for implementation
Following link for integrate redmine with keycloak**
[https://github.com/devopskube/redmine_openid_connect](https://github.com/devopskube/redmine_openid_connect)

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image55.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image55.png)

**Go to the realm roles section**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image44.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image44.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image23.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image23.png)

**Also Creating Gamma Role**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image31.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image31.png)

**Creating User Role**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image28.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image28.png)

**User creating**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image35.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image35.png)

**Also set password for every user**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image8.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image8.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image6.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image6.png)

**Assign the role to given user**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image48.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image48.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image18.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image18.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image20.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image20.png)

<aside>
💡 **And now give it roles
Here ravi have two roles one is gamma and another is User role**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image58.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image58.png)

<aside>
💡 **Create a client scope named openid in Clinet_Scopes -**

</aside>

<aside>
💡

**Create a client scope named openid in Clinet_Scopes -**

**Add the following mappers -**

**username -- of type user property -- token claim name should be - user_name**

**member_of -- of type user realm role -- token claim name should be member_of**

**roles --- of type user realm role -- token claim name should be roles**

**add all these mappers to id_token , userinfo and access_token.**

</aside>

<aside>
💡 **NOW create client scop named= openid**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image16.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image16.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image16.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image16.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image9.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image9.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image7.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image7.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image22.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image22.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image26.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image26.png)

<aside>
💡 **Now we are total 3 mapper**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image3.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image3.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image42.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image42.png)

**Now assign the roles to scope**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image25.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image25.png)

<aside>
💡 **Now go to client section search for**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image21.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image21.png)

<aside>
💡 **Add client scope name open id as we previously created**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image57.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image57.png)

<aside>
💡 **Go to Realm setting section and disable the ssl**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image61.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image61.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image46.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image46.png)

<aside>
💡 **Now login in redmine first**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image59.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image59.png)

<aside>
💡 **Now go for plugin section**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image38.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image38.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image2.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image2.png)

**Chek your ream url from Realm Setting in keycloak**

**Click on Ralm Setting > OpeID Endpoint Configuration**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image36.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image36.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image34.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image34.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image47.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image47.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image30.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image30.png)

<aside>
💡 **When i Click the single sign On**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image37.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image37.png)

<aside>
💡 **Now I am going to login with my manoj user which have Admin role**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image40.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image40.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image53.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image53.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image19.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image19.png)

<aside>
💡 **Also I am going to login with another user which is ravi and he is a normal user**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image17.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image17.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image60.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image60.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image5.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image5.png)

<aside>
💡 **And then log out**

</aside>

## Integrate Superset with keycloak

<aside>
💡 **Superset Dockerfile**

</aside>

```bash
**manoj@crunchy-postgres:~$ cat Dockerfile**

```

```bash
**FROM apache/superset
# Switching to root to install the required packages
USER root
# if you prefer Postgres, you may want to use `psycopg2-binary` instead
RUN pip install psycopg2-binary
RUN pip install flask-oidc==1.3.0
RUN pip install flask_openid
RUN pip install python-keycloak
RUN pip install itsdangerous==2.0.1
# Switching back to using the `superset` user
USER superset**
```

```bash

```

---

**Build this image with tag**

```bash
**manoj@crunchy-postgres:~$ podman build -t superset:manoj .
STEP 1/8: FROM apache/superset
STEP 2/8: USER root
--> Using cache 9f0efc66f1c77f0f265e54d60ecd6b3cfe0008b5edc1f75f6d154cc27f18a2ec
--> 9f0efc66f1c
STEP 3/8: RUN pip install psycopg2-binary
--> Using cache b88bb033dda729ae992c2fee56d237c4b555fdc2beb5e8f22697a73882100c02
--> b88bb033dda
STEP 4/8: RUN pip install flask-oidc==1.3.0
--> Using cache a1497a4553fcb3711db32695fb1c1796a12977e6be12bb4da8b6ee8169d1e436
--> a1497a4553f
STEP 5/8: RUN pip install flask_openid
--> Using cache cc85c687110d41c157a64e4e96a3fb0376d58971e6c574cf136a5397017238c0
--> cc85c687110
STEP 6/8: RUN pip install python-keycloak
--> Using cache 8537ebfb684174f69952b7da8125bef2fde6f7c1ecdcc71a02ac2f86e8fae77f
--> 8537ebfb684
STEP 7/8: RUN pip install itsdangerous==2.0.1
--> Using cache d170cde073db53e3d65f49d24b9e39def15295e65806f19fd3cc6f572427fa96
--> d170cde073d
STEP 8/8: USER superset
--> Using cache 37735fa096a458d96e7c84f71350197f8dffc5814d9d471ec12196539525c8f1
COMMIT superset:manoj
--> 37735fa096a
Successfully tagged localhost/superset:manoj
Successfully tagged localhost/superset_apache:mj
37735fa096a458d96e7c84f71350197f8dffc5814d9d471ec12196539525c8f1
manoj@crunchy-postgres:~$**
```

---

```bash
**manoj@crunchy-postgres:~$ podman images
REPOSITORY TAG IMAGE ID CREATED SIZE
localhost/superset manoj 37735fa096a4 4 days ago 1.33 GB**
```

---

```bash
**manoj@crunchy-postgres:~$ mkdir -p superset/app**
```

**Then run the superset container with volume**

**Run this command for generating secret key**

```bash
**manoj@crunchy-postgres:~$ openssl rand -base64 42
0s3Hcl9cqUPBvWZdULVNygH2gjmn44/JFX//HhQXDGqYwSYlufMAXq0O**
```

---

```bash
**manoj@crunchy-postgres:~$ podman run -d -p 8088:8088 -v /home/manoj/superse/app:/app/pythonpath -e "SUPERSET_SECRET_KEY=0s3Hcl9cqUPBvWZdULVNygH2gjmn44/JFX//HhQXDGqYwSYlufMAXq0O" --name superHIT superset:manoj
26890be963c6a3ddaa929466b32c921115e556e70432f2ebec6ae910fb376416
manoj@crunchy-postgres:~$**
```

---

**Go on browser and type http://<ip>:8088**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image12.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image12.png)

```bash
**manoj@crunchy-postgres:~$ podman exec -it superHIT superset fab create-admin --username admin --firstname Superset --lastname Admin --email admin@superset.com --password admin
logging was configured successfully
2023-10-10 08:45:45,888:INFO:superset.utils.logging_configurator:logging was configured successfully
2023-10-10 08:45:45,897:INFO:root:Configured event logger of type <class 'superset.utils.log.DBEventLogger'>
/usr/local/lib/python3.9/site-packages/flask_limiter/extension.py:293: UserWarning: Using the in-memory storage for tracking rate limits as no storage was explicitly specified. This is not recommended for production use. See: https://flask-limiter.readthedocs.io#configuring-a-storage-backend for documentation about configuring the storage backend.
warnings.warn(
Recognized Database Authentications.
Admin User admin created.**
```

---

**Migrate local DB to latest**

```bash
**manoj@crunchy-postgres:~$ podman exec -it superHIT superset db upgrade
logging was configured successfully
2023-10-10 08:48:23,761:INFO:superset.utils.logging_configurator:logging was configured successfully
2023-10-10 08:48:23,767:INFO:root:Configured event logger of type <class 'superset.utils.log.DBEventLogger'>
/usr/local/lib/python3.9/site-packages/flask_limiter/extension.py:293: UserWarning: Using the in-memory storage for tracking rate limits as no storage was explicitly specified. This is not recommended for production use. See: https://flask-limiter.readthedocs.io#configuring-a-storage-backend for documentation about configuring the storage backend.
warnings.warn(
WARNI [alembic.env] SQLite Database support for metadata databases will be removed in a future version of Superset.
INFO [alembic.runtime.migration] Context impl SQLiteImpl.
INFO [alembic.runtime.migration] Will assume transactional DDL.**
```

---

**Setup roles**

```bash
**manoj@crunchy-postgres:~$ podman exec -it superHIT superset init
logging was configured successfully**
```

---

<aside>
💡 **After done this steps restart superset you will be able to login with admin user with password (admin)
Go inside the container with root privileges and update it and install vim for some changes**

</aside>

<aside>
💡 **> TALISMAN_ENABLED defaults to True; set this to False in order to disable CSP**

</aside>

<aside>
💡 **Make changes in config.py in superset directory inside the container**

</aside>

```bash
**manoj@crunchy-postgres:~$ podman exec -it -u root superHIT bash

root@26890be963c6:/app# apt-get update
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [52.1 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:4 http://deb.debian.org/debian bookworm/main amd64 Packages [8780 kB]
Get:5 http://deb.debian.org/debian bookworm-updates/main amd64 Packages [6408 B]
Get:6 http://deb.debian.org/debian-security bookworm-security/main amd64 Packages [78.6 kB]
Fetched 9116 kB in 3s (3405 kB/s)
Reading package lists... Done

root@26890be963c6:/app# apt install vim
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:**
```

```bash
r**oot@26890be963c6:/app/superset# vim config.py**
```

---

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image45.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image45.png)

<aside>
💡 **After making changes restart the container**

</aside>

```bash
**manoj@crunchy-postgres:~$ podman restart superHIT
26890be963c6a3ddaa929466b32c921115e556e70432f2ebec6ae910fb376416**
```

---

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image11.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image11.png)

**Go to setting section and add postgres database which was already running**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image29.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image29.png)

**Select postgres in my case**

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image32.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image32.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image24.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image24.png)

<aside>
💡 **NOW database has connected successfully,
ALL THE ABOVE STEPS ARE TAKEN FOR ONLY LOGIN WITH LOCAL Admin
NOW we are going toward the implement superset with keycloak**

</aside>

<aside>
💡 **Reference link:-
[https://github.com/apache/superset/discussions/13915](https://github.com/apache/superset/discussions/13915)**

</aside>

**NOW I HAVE CREATED 3 files inside app directory**

```bash
**manoj@crunchy-postgres:~/superse/app$ ls
client_secret.json keycloak_security_manager.py superset_config.py

manoj@crunchy-postgres:~/superse/app$ vim client_secret.json
manoj@crunchy-postgres:~/superse/app$ cat client_secret.json**
```

```bash

**{
    "web": {
        "issuer": "http://192.168.122.199:8080/realms/Keenable",
        "auth_uri": "http://192.168.122.199:8080/realms/Keenable/protocol/openid-connect/auth",
        "client_id": "keenable",
        "client_secret": "aGwAujXoaLO9AFWyYSitPSdNESn77e5r",
        "verify_ssl_server": false,
        "redirect_uris": ["http://192.168.122.199:8088/*"],
        "userinfo_uri": "http://192.168.122.199:8080/realms/Keenable/protocol/openid-connect/userinfo",
        "token_uri": "http://192.168.122.199:8080/realms/Keenable/protocol/openid-connect/token",
        "token_introspection_uri": "http://192.168.122.199:8080/realms/Keenable/protocol/openid-connect/introspect"
    }
}**
```

---

```bash
**manoj@crunchy-postgres:~/superse/app$ vim keycloak_security_manager.py**
```

```bash
**manoj@crunchy-postgres:~/superse/app$ cat keycloak_security_manager.py**
```

```bash
import os
from flask_appbuilder.security.manager import AUTH_OID
from superset.security import SupersetSecurityManager
from flask_oidc import OpenIDConnect
from flask_login import login_user
from urllib.parse import quote
from flask import redirect, request
from flask_appbuilder.security.views import AuthOIDView
from flask_appbuilder.views import expose
import logging

logger = logging.getLogger(__name__)

class OIDCSecurityManager(SupersetSecurityManager):

    def __init__(self, appbuilder):
        super(OIDCSecurityManager, self).__init__(appbuilder)
        if self.auth_type == AUTH_OID:
            self.oid = OpenIDConnect(self.appbuilder.get_app)
        self.authoidview = AuthOIDCView

class AuthOIDCView(AuthOIDView):

    @expose('/login/', methods=['GET', 'POST'])
    def login(self, flag=True):
        sm = self.appbuilder.sm
        oidc = sm.oid
        superset_roles = ["Admin", "Alpha", "Gamma", "Public", "granter", "sql_lab"]
        default_role = "Gamma"

        @self.appbuilder.sm.oid.require_login
        def handle_login():
            user = sm.auth_user_oid(oidc.user_getfield('email'))

            if user is None:
                info = oidc.user_getinfo(['preferred_username', 'given_name', 'family_name', 'email', 'roles'])
                roles = [role for role in superset_roles if role in info.get('roles', [])]
                roles += [default_role, ] if not roles else []
                user = sm.add_user(info.get('preferred_username'), info.get('given_name'), info.get('family_name'),
                                   info.get('email'), [sm.find_role(role) for role in roles])

            login_user(user, remember=False)
            return redirect(self.appbuilder.get_url_for_index)

        return handle_login()

    @expose('/logout/', methods=['GET', 'POST'])
    def logout(self):
        oidc = self.appbuilder.sm.oid

        oidc.logout()
        super(AuthOIDCView, self).logout()
        redirect_url = 'http://192.168.122.199:8088/login/'
        logging.debug('This is redirect uri %s ',self.appbuilder.get_url_for_login)
        return redirect(oidc.client_secrets.get('issuer') + '/protocol/openid-connect/logout?client_id='+ oidc.client_secrets.get('client_id')+'&post_logout_redirect_uri=' + quote(redirect_url))

****
```

---

**manoj@crunchy-postgres:~/superse/app$ vim superset_config.py**

```bash
**manoj@crunchy-postgres:~/superse/app$ cat superset_config.py**
```

```bash

**from keycloak_security_manager import OIDCSecurityManager
from flask_appbuilder.security.manager import AUTH_OID
import os

# Get current directory
curr = os.path.abspath(os.getcwd())

# Superset configurations for OIDC authentication and session management
AUTH_TYPE = AUTH_OID
#SECRET_KEY = 'U66tVZgRFbQXTarU5mf/IYw2wpTOLZ8Tz6r6kSxYw732HqhzBbjiRokf'
OIDC_CLIENT_SECRETS = curr + '/pythonpath/client_secret.json'
OIDC_ID_TOKEN_COOKIE_SECURE = False
OIDC_REQUIRE_VERIFIED_EMAIL = False
OIDC_OPENID_REALM = 'Keenable'
OIDC_INTROSPECTION_AUTH_METHOD = 'client_secret_post'
CUSTOM_SECURITY_MANAGER = OIDCSecurityManager
AUTH_USER_REGISTRATION = False
AUTH_USER_REGISTRATION_ROLE = 'Gamma'
OIDC_TOKEN_TYPE_HINT = 'access_token'
OVERWRITE_REDIRECT_URI = 'http://192.168.122.199:8088/oidc_callback'**
```

---

**Again update DB after making changes**

```bash
**manoj@crunchy-postgres:~/superse/app$ podman exec -it superHIT superset db upgrade

Loaded your LOCAL configuration at [/app/pythonpath/superset_config.py]
logging was configured successfully
2023-10-10 09:46:12,231:INFO:superset.utils.logging_configurator:logging was configured successfully
2023-10-10 09:46:12,239:INFO:root:Configured event logger of type <class 'superset.utils.log.DBEventLogger'>
/usr/local/lib/python3.9/site-packages/flask_limiter/extension.py:293: UserWarning: Using the in-memory storage for tracking rate limits as no storage was explicitly specified. This is not recommended for production use. See: https://flask-limiter.readthedocs.io#configuring-a-storage-backend for documentation about configuring the storage backend.
warnings.warn(
WARNI [alembic.env] SQLite Database support for metadata databases will be removed in a future version of Superset.
INFO [alembic.runtime.migration] Context impl SQLiteImpl.
INFO [alembic.runtime.migration] Will assume transactional DDL.
manoj@crunchy-postgres:~/superse/app$**
```

---

```bash
**manoj@crunchy-postgres:~/superse/app$ podman exec -it superHIT superset init

Loaded your LOCAL configuration at [/app/pythonpath/superset_config.py]
logging was configured successfully
2023-10-10 09:48:34,301:INFO:superset.utils.logging_configurator:logging was configured successfully
2023-10-10 09:48:34,309:INFO:root:Configured event logger of type <class 'superset.utils.log.DBEventLogger'>
/usr/local/lib/python3.9/site-packages/flask_limiter/extension.py:293: UserWarning: Using the in-memory storage for tracking rate limits as no storage was explicitly specified. This is not recommended for production use. See: https://flask-limiter.readthedocs.io#configuring-a-storage-backend for documentation about configuring the storage backend.
warnings.warn(
Syncing role definition
2023-10-10 09:48:38,015:INFO:superset.security.manager:Syncing role definition
Syncing Admin perms
2023-10-10 09:48:38,054:INFO:superset.security.manager:Syncing Admin perms
Syncing Alpha perms
2023-10-10 09:48:38,322:INFO:superset.security.manager:Syncing Alpha perms
Syncing Gamma perms
2023-10-10 09:48:38,605:INFO:superset.security.manager:Syncing Gamma perms
Syncing sql_lab perms
2023-10-10 09:48:38,937:INFO:superset.security.manager:Syncing sql_lab perms**
```

---

<aside>
💡 **Also restart the container again
(Remember after making changes you will have to restart the container)**

</aside>

```bash
**manoj@crunchy-postgres:~/superse/app$ podman restart superHIT
26890be963c6a3ddaa929466b32c921115e556e70432f2ebec6ae910fb376416**
```

---

<aside>
💡 **Hit on browser with http:<ip>:8088 it will automatically redirect toward keycloak sign in**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image52.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image52.png)

<aside>
💡 **Login with mano user which have admin credentials as I have given in keycloak configuration**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image14.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image14.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image39.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image39.png)

<aside>
💡 **LIST OF USERS AS I HAVE CONFIGURED IN KEYCLOAK**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image33.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image33.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image62.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image62.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image54.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image54.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image56.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image56.png)

<aside>
💡 **After logout check another simple user are able to login or not
NOW LOGIN WITH SIMPLE USERS WHO HAVE GAMMA ROLE**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image41.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image41.png)

<aside>
💡 **Yes we are able to login with second users also**

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image27.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image27.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image15.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image15.png)

<aside>
💡 **Gamma**: Gamma users have restricted access and can only consume data from assigned data sources. They can view slices and dashboards created from these data sources.

</aside>

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image4.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image4.png)

![Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image10.png](Implementing%20Keycloak%20SSO%20for%20both%20Redmine%20and%20Sup%2039d66d257b8548068545de900504b10f/image10.png)