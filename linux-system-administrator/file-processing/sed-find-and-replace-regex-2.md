# Sed File Processing 2

## Task

> The backup server in the Stratos DC contains several template XML files used by the Nautilus application. However, these template XML files must be populated with valid data before they can be used. One of the daily tasks of a system admin working in the xFusionCorp industries is to apply string and file manipulation commands!<br><br>Replace all occurances of the string '*String*' to '*Torpedo*' on the XML file `/root/nautilus.xml` located in the backup server.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* None.

## Steps

```bash
# Connect to the first app server
ssh clint@stbkp01

# Get root
sudo -i

```bash
# Get root
sudo -i

# Create a back up
cd /root
cp -ar /root/nautilus.xml /root/nautilus.xml.bak

# Replace 'String' with 'Torpedo'
sed -i -r 's/String/Torpedo/g' /root/nautilus.xml

# Check changes.
diff /root/nautilus.xml /root/nautilus.xml.bak
```

```
10c10
<       <string>Torpedo</string>
---
>       <string>String</string>
...
```

```bash
# Delete the backup
rm -rf /root/nautilus.xml.bak
```