# Exercise 11 - OPTIONAL: Discovering the Tower API

You have used the Tower API a couple of times in this lab already. In
this chapter we’ll describe two ways to discover the Tower API if you
need to dive in deeper. While the [principles of the Tower
API](https://docs.ansible.com/ansible-tower/latest/html/towerapi/index.html)
are documented and there is an [API reference
guide](https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html#/),
it’s often more efficient to just browse and discover the API.

## Browsing and Using the Tower API interactively

The Tower API is browsable, which means you can just click your way
through it:

1.  Go to the Tower UI in your browser and make sure you’re logged in as
    admin.

2.  Replace the end of the URL with `/api` e.g.
    `https://tower2-<GUID>.rhpds.opentlc.com/api`

3.  You’re now in the API, notice that there are two versions. v1 will
    be retired soon so go to v2.

4.  While in `/api/v2`:

      - you see a list of clickable object types

      - on the right upper side, there is a button **OPTIONS** which
        tells you what you can do with the current object in terms of
        API.

      - next to it there is a **GET** button which allows you to choose
        between getting the (raw or not) JSON output or the API format,
        which you’re currently admiring by default.

5.  Click on the `/api/v2/users/` link and discover some more features:

      - There is a list of all objects of the given type

      - Each individual object can be reached using the `url` field
        ("url": "/api/v2/users/1/",)

      - Most objects have a `related` field, which allows you to jump
        from object to object

      - At the bottom of the page, there is a new field which allows you
        to *post* a new object, so let’s do this and create a new user
        name John Smith (user name doesn’t matter)


> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
> The JSON should roughly look like this:
>
>     {
>         "username": "jsmith",
>         "first_name": "John",
>         "last_name": "Smith",
>         "email": "jsmith@example.com",
>         "is_superuser": false,
>         "is_system_auditor": false,
>         "password": "redhat"
>     }
>
> and the result should be a 201 telling you about your success. You can
> log-in with the password and see that you see… nothing, because you have
> no rights.
> </p>
> </details>

Now log in again as admin and go back to the list of users:
**https://tower2-&lt;GUID&gt;.rhpds.opentlc.com/api/v2/users/**

  - Click on the **url** field of your new friend John Smith and notice
    a few more things:

      - There is a red **DELETE** button at the top right level. Guess
        for what?

      - At the bottom of the page, the dialog shows **PUT** and
        **PATCH** buttons.

So why not patch the user to be named "Johnny" instead of "John"?

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
> Add this to the **CONTENT** field:
>
>     {
>         "first_name": "Johnny"
>     }
>
> And press the **PATCH** button.
> </p>
> </details>

Now try to **PUT** **last\_name** "Smithy" using the same approach. What
happens?

> **Warning**
>
> <details><summary>Solution below!</summary>
> <p>
> Enter this into the **CONTENT** field and press **PUT**:
>
>     {
>         "last_name": "Smithy"
>     }
>
> This will fail. In the case of **PUT** you need to enter all mandatory
> fields, even if you don’t want to modify them:
>
>     {
>         "username": "jsmith",
>         "last_name": "Smithy"
>     }
> </p>
> </details>

When you’re done press the red **DELETE** button and remove Johnny
Smithy.

## Using awx to Learn the API

The Web UI is nice but we love the command line, right? To learn about
API calls `awx` comes to the rescue. For the next steps bring up an
SSH session and make sure you are user root on **control.example.com**.

Let’s start simple and try to get the version of Tower installed:

    [root@ansible ~]# awx version --verbose
    Tower CLI 3.3.0
    API v2
    GET https://tower2.example.com/api/v2/config/
    Params: {}

    Ansible Tower 3.4.1
    Ansible 2.7.5

You see that with the `--verbose` option, awx tells us which API
calls it’s making, what **parameters** it’s sending with **GET**
requests and what **data** is needed for **POST** actions.

In this simple case you can simply take the call and run it with e.g.
**curl**:

    [root@ansible ~]# curl -k -H 'Content-Type: application/json' --user admin:r3dh4t1! \
            --data '{}' \
            -X GET https://tower1.example.com/api/v2/config/ | jq

> **Tip**
>
> `jq` is optional but useful for us humans to understand the output
> without getting dizzy… in this case it comes from the EPEL repo. If
> you don’t have `jq` appending `| python -m json.tool` to the command
> is better then nothing.

### Practice, Practice…

Using `awx` to learn about the API call and executing it via curl
e.g. in scripts is really useful so let’s practice a bit. What about
creating a new user, say Albert Miller?

> **Tip**
>
> Consider that the parameters shown by awx are in Python format
> (single quotes and the unicode `u`) but we need to send data in JSON
> format (double quotes).

First create the user with awx, then delete it again. Use
`--verbose` to get the API invocation.

    [root@ansible ~]# awx user create --username amiller --email amiller@example.com --password redhat --verbose

    *** DETAILS: Checking for an existing record. *********************************
    GET https://tower2.example.com/api/v2/users/
    Params: {'username': u'amiller'}

    *** DETAILS: Writing the record. **********************************************
    POST https://tower2.example.com/api/v2/users/
    Data: {'username': u'amiller', 'password': u'redhat', 'email': u'amiller@example.com'}

    [root@ansible ~]# awx user delete --username amiller --verbose

    *** DETAILS: Getting the record. **********************************************
    GET https://tower2.example.com/api/v2/users/
    Params: {'username': u'amiller'}

    DELETE /users/3/
    DELETE https://tower2.example.com/api/v2/users/3/

Now we’ll do the same using **curl** with the API endpoints, parameters
and data we have learned from `awx`:

> **Warning**
>
> The "Getting the record" is (sadly) a bit misleading… you need to add
> `?username=amiller` to filter on the username:

Check if the user exists:

    [root@ansible ~]# curl -k -H 'Content-Type: application/json' --user admin:r3dh4t1! \
            -X GET https://tower1.example.com/api/v2/users/?username=amiller

Once you’ve found out that the user doesn’t exist by **count:0** in the
reply, you can create it:

    [root@ansible ~]# curl -k -H 'Content-Type: application/json' --user admin:r3dh4t1! \
            --data '{"username": "amiller", "password": "redhat", "email": "amiller@example.com"}' \
            -X POST https://tower1.example.com/api/v2/users/?username=amiller

Run the `curl` command from above again to check the user now exists, it
should return **count:1** and the user’s data.

Note the ID of the user and then delete it:

> **Warning**
>
> Replace **&lt;ID&gt;**

    [root@ansible ~]# curl -k -H 'Content-Type: application/json' --user admin:r3dh4t1! \
            -X DELETE https://tower1.example.com/api/v2/users/<ID>/ #

  - don’t forget the slash at the end of the URL, favorite error\!

&lt;/div&gt;&lt;/details&gt;
