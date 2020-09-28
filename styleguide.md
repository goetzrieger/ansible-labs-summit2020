# Style guide for lab guides in this repository

Our style guidelines ensure that all labs follow a consistent layout - and make it easier for contributors to edit and improve the labs.

## Headers

Headers are marked with hash signs:

```
# Document Title (Level 0)

## Level 1 Section Title

### Level 2 Section Title
```

## Formatted text, highlights, etc.

Text will be standard formatted (no italics, no bold, etc.), except for the following situations.

### Tips and warnings

We use Hugo short codes to format tips and warnings. We do not use info or note! Use the following example:

```
{{% notice tip %}}
This is a tip
{{% /notice %}}
```

```
{{% notice warning %}}
This is a warning
{{% /notice %}}
```

For example solutions, we use the collapse feature:

```
    <details><summary>**Click here for Solution**</summary>
    <p>
    ```bash
    [student<X>@ansible-1 ~]$ ansible-doc -l | grep -i dnf
    [student<X>@ansible-1 ~]$ ansible-doc dnf
    [student<X>@ansible-1 ~]$ ansible all -m dnf -a 'name=vim state=latest' -b
    ```
    </p>
    </details>
```

### Code, addresses, host names, everything for copy & paste

Every piece of text which in theory can be copied and pasted during the lab, is marked as `code` with back tics:

```
`curl https://www.redhat.com`
`admin@example.com`
```

If the syntax formatter doesn't automatically recognize your code, add it, e.g. \`\`\`bash, \`\`\`yaml, etc.

### Visual elements

In these labs the users often have to interact with UIs, and thus pieces of the UI need to be recognized. Such elements are marked **bold** and are spelled and written exactly the same way they are in the UI. For example, if the lab requires that you  click on the tab called projects, written in all capital letters, the lab guide would refer to it as **PROJECTS**. The same rule applies to command outputs, which are also **BOLD**.

```
Click on **INVENTORY** and afterwards note the **TYPE** column with the key word **Inventory**.
```
