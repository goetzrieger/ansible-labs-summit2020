+++
title = "Discovering the Tower API"
weight = 11
+++

## OPTIONAL EXERCISE

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

2.  Replace the end of the URL with `/api` e.g. `https://{{< param "external_tower" >}}/api`

3.  There is currently only one API valid, so while in `/api/v2`:

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


<details><summary>**Click here for Solution**</summary>
<p>
The JSON should roughly look like this:
```JSON
    {
        "username": "jsmith",
        "first_name": "John",
        "last_name": "Smith",
        "email": "jsmith@example.com",
        "is_superuser": false,
        "is_system_auditor": false,
        "password": "redhat"
    }
```
and the result should be a 201 telling you about your success. You can
log-in with the password and see that you see… nothing, because you have
no rights.
</p>
</details>

Now log in again as admin and go back to the list of users:
`https://{{< param "external_tower" >}}/api/v2/users/`

  - Click on the **url** field of your new friend John Smith and notice
    a few more things:

      - There is a red **DELETE** button at the top right level. Guess
        for what?

      - At the bottom of the page, the dialog shows **PUT** and
        **PATCH** buttons.

So why not patch the user to be named "Johnny" instead of "John"?

<details><summary>**Click here for Solution**</summary>
<p>
Add this to the **CONTENT** field:
```JSON
    {
        "first_name": "Johnny"
    }
```
And press the **PATCH** button.
</p>
</details>

Now try to **PUT** **last\_name** "Smithy" using the same approach. What
happens?

<details><summary>**Click here for Solution**</summary>
<p>
Enter this into the **CONTENT** field and press **PUT**:
```JSON
    {
        "last_name": "Smithy"
    }
```
This will fail. In the case of **PUT** you need to enter all mandatory
fields, even if you don’t want to modify them:
```JSON
    {
        "username": "jsmith",
        "last_name": "Smithy"
    }
```
</p>
</details>

When you’re done press the red **DELETE** button and remove Johnny
Smithy.
