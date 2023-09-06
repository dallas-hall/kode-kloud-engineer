# Find Files

## Task

> During a routine security audit, the team identified an issue on the Nautilus App Server. Some malicious content was identified within the website code. After digging into the issue they found that there might be more infected files. Before doing a cleanup they would like to find all similar files and copy them to a safe location for further investigation. Accomplish the task as per the following requirements:
>
> a. On App Server 3 at location `/var/www/html/news` find out all files (not directories) having `.php` extension.
>
> b. Copy all those files along with their parent directory structure to location `/news` on same server.
>
> c. Please make sure not to copy the entire `/var/www/html/news` directory content.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://unix.stackexchange.com/questions/83593/copy-specific-file-type-keeping-the-folder-structure & `man find`

## Steps

```bash
# Connect to application servers
ssh banner@stapp03

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Switch to root
sudo -i

# Copy all the .php files with their folder structure to /new
find /var/www/html/news -type f -iname '*.php' -exec cp --parents '{}' /news ';'

# Install tree
dnf install -y tree
```

```
...
Installed:
  tree-1.7.0-15.el8.x86_64
...
```

```bash
# Check results
tree /news/
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
/news
`-- var
    `-- www
        `-- html
            `-- news
                |-- index.php
                |-- wp-activate.php
                |-- wp-admin
                |   |-- about.php
                |   |-- admin-ajax.php
                |   |-- admin-footer.php
                |   |-- admin-functions.php
                |   |-- admin-header.php
                |   |-- admin-post.php
                |   |-- admin.php
                |   |-- async-upload.php
                |   |-- authorize-application.php
                |   |-- comment.php
                |   |-- credits.php
                |   |-- custom-background.php
                |   |-- custom-header.php
                |   |-- customize.php
                |   |-- edit-comments.php
                |   |-- edit-form-advanced.php
                |   |-- edit-form-blocks.php
                |   |-- edit-form-comment.php
                |   |-- edit-link-form.php
                |   |-- edit-tag-form.php
                |   |-- edit-tags.php
                |   |-- edit.php
                |   |-- erase-personal-data.php
                |   |-- export-personal-data.php
                |   |-- export.php
                |   |-- freedoms.php
                |   |-- import.php
                |   |-- includes
                |   |   |-- admin-filters.php
...
```

</details>

We are done.