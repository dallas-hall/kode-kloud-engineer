# File Permissions

## Task

> During a routine security audit, the team identified an issue on the Nautilus App Server. Some malicious content was identified within the website code. After digging into the issue they found that there might be more infected files. Before doing a cleanup they would like to find all similar files and copy them to a safe location for further investigation. Accomplish the task as per the following requirements:<br><br>a. On App Server 1 at location `/var/www/html/official` find out all files (not directories) having `.css` extension.<br>b. Copy all those files along with their parent directory structure to location `/official` on same server.<br>c. Please make sure not to copy the entire `/var/www/html/official` directory content.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* https://unix.stackexchange.com/questions/83593/copy-specific-file-type-keeping-the-folder-structure & `man find`

## Steps

```bash
# Connect to application servers
ssh tony@stapp01

# Check current Linux version, it was CentOS 7.6
cat /etc/*release*

# Switch to root
sudo -i

# Copy all the .css files with their folder structure to /official
find /var/www/html/official -type f -iname '*.css' -exec cp --parents '{}' /official ';'

# Check results
tree /official/
```

```
/official/
└── var
    └── www
        └── html
            └── official
                ├── wp-admin
                │   └── css
                │       ├── about.css
                │       ├── about.min.css
                │       ├── about-rtl.css
                │       ├── about-rtl.min.css
                │       ├── admin-menu.css
                │       ├── admin-menu.min.css
                │       ├── admin-menu-rtl.css
                │       ├── admin-menu-rtl.min.css
                │       ├── code-editor.css
                │       ├── code-editor.min.css
                │       ├── code-editor-rtl.css
                │       ├── code-editor-rtl.min.css
                │       ├── color-picker.css
                │       ├── color-picker.min.css
                │       ├── color-picker-rtl.css
                │       ├── color-picker-rtl.min.css
                │       ├── colors
                │       │   ├── blue
                │       │   │   ├── colors.css
                │       │   │   ├── colors.min.css
                │       │   │   ├── colors-rtl.css
                │       │   │   └── colors-rtl.min.css
...
```