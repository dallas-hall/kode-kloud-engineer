# Sed File Processing

## Task

> There is some data on Nautilus App Server 1 in Stratos DC. Data needs to be altered in several of the files. On Nautilus App Server 1, alter the `/home/BSD.txt` file as per details given below:<br><br>a. Delete all lines containing word *copyright* and save results in `/home/BSD_DELETE.txt` file. (Please be aware of case sensitivity)<br><br>b. Replace all occurrence of word *and* to *is* and save results in `/home/BSD_REPLACE.txt` file.<br><br>**Note:** Let's say you are asked to replace word *to* with *from*. In that case, make sure not to alter any words containing this string; for example up**to**, contribu**to**r etc.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Delete lines matching a string - https://stackoverflow.com/questions/5410757/how-to-delete-from-a-text-file-all-lines-that-contain-a-specific-string
* Word bounadries - https://unix.stackexchange.com/questions/184815/how-can-i-find-and-replace-only-if-a-match-forms-a-whole-word

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Get root
sudo -i

# Create a back up
cd /home
cp -ar BSD.txt BSD.txt.bak

# Delete all lines containing the string 'copyright'
sed -r '/copyright/d' BSD.txt > BSD_DELETE.txt

# Check changes.
diff BSD.txt BSD_DELETE.txt
```

```
< Copyright (c) <year>, <copyright holder>
6d4
< 1. Redistributions of source code must retain the above copyright
8d5
< 2. Redistributions in binary form must reproduce the above copyright
29d25
< Copyright (c) <year>, <copyright holder>
34d29
< 1. Redistributions of source code must retain the above copyright
36d30
< 2. Redistributions in binary form must reproduce the above copyright
58d51
< Copyright (c) <year>, <copyright holder>
63d55
< 1. Redistributions of source code must retain the above copyright
65d56
< 2. Redistributions in binary form must reproduce the above copyright
87d77
< Copyright (c) <year>, <copyright holder>
92d81
< 1. Redistributions of source code must retain the above copyright
94d82
< 2. Redistributions in binary form must reproduce the above copyright
116d103
< Copyright (c) <year>, <copyright holder>
121d107
< 1. Redistributions of source code must retain the above copyright
123d108
< 2. Redistributions in binary form must reproduce the above copyright
144d128
< Copyright (c) <year>, <copyright holder>
149d132
< 1. Redistributions of source code must retain the above copyright
151d133
< 2. Redistributions in binary form must reproduce the above copyright
```

```bash
# Replace 'and' to 'is
sed -r 's/\band\b/is/g' BSD.txt > BSD_REPLACE.txt

# Check changes.
diff BSD.txt BSD_REPLACE.txt
```

```
4c4
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
7c7
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
9,10c9,10
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
32c32
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
35c35
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
37,38c37,38
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
61c61
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
64c64
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
66,67c66,67
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
90c90
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
93c93
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
95,96c95,96
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
119c119
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
122c122
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
124,125c124,125
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
147c147
< Redistribution and use in source and binary forms, with or without
---
> Redistribution is use in source is binary forms, with or without
150c150
<    notice, this list of conditions and the following disclaimer.
---
>    notice, this list of conditions is the following disclaimer.
152,153c152,153
<    notice, this list of conditions and the following disclaimer in the
<    documentation and/or other materials provided with the distribution.
---
>    notice, this list of conditions is the following disclaimer in the
>    documentation is/or other materials provided with the distribution.
```

```bash
# Delete the backup
rm -rf /home/BSD.txt.bak
```