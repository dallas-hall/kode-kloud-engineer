# Sed File Processing

## Task

> There is some data on Nautilus App Server 1 in Stratos DC. Data needs to be altered in several of the files. On Nautilus App Server 1, alter the `/home/BSD.txt` file as per details given below:
>
> a. Delete all lines containing word *following* and save results in `/home/BSD_DELETE.txt` file. (Please be aware of case sensitivity)
>
> b. Replace all occurrence of word *and* to *is* and save results in `/home/BSD_REPLACE.txt` file.
>
> **Note:** Let's say you are asked to replace word *to* with *from*. In that case, make sure not to alter any words containing this string; for example up**to**, contribu**to**r etc.

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

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Create a back up
cd /home
cp -ar BSD.txt BSD.txt.bak

# Delete all lines containing the string 'software'
sed -r '/following/d' BSD.txt > BSD_DELETE.txt

# Check changes.
diff BSD.txt BSD_DELETE.txt
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
5d4
< modification, are permitted provided that the following conditions are met:
7d5
<    notice, this list of conditions and the following disclaimer.
9d6
<    notice, this list of conditions and the following disclaimer in the
12d8
<    must display the following acknowledgement:
33d28
< modification, are permitted provided that the following conditions are met:
35d29
<    notice, this list of conditions and the following disclaimer.
37d30
<    notice, this list of conditions and the following disclaimer in the
40d32
<    must display the following acknowledgement:
62d53
< modification, are permitted provided that the following conditions are met:
64d54
<    notice, this list of conditions and the following disclaimer.
66d55
<    notice, this list of conditions and the following disclaimer in the
69d57
<    must display the following acknowledgement:
91d78
< modification, are permitted provided that the following conditions are met:
93d79
<    notice, this list of conditions and the following disclaimer.
95d80
<    notice, this list of conditions and the following disclaimer in the
98d82
<    must display the following acknowledgement:
120d103
< modification, are permitted provided that the following conditions are met:
122d104
<    notice, this list of conditions and the following disclaimer.
124d105
<    notice, this list of conditions and the following disclaimer in the
127d107
<    must display the following acknowledgement:
148d127
< modification, are permitted provided that the following conditions are met:
150d128
<    notice, this list of conditions and the following disclaimer.
152d129
<    notice, this list of conditions and the following disclaimer in the
155d131
<    must display the following acknowledgement:
```

</details>

```bash
# Replace 'and' to 'their'
sed -r 's/\bor\b/for/g' BSD.txt > BSD_REPLACE.txt

# Check changes.
diff BSD.txt BSD_REPLACE.txt
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```
4c4
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
10,11c10,11
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
15c15
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
32c32
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
38,39c38,39
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
43c43
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
61c61
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
67,68c67,68
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
72c72
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
90c90
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
96,97c96,97
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
101c101
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
119c119
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
125,126c125,126
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
130c130
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
147c147
< Redistribution and use in source and binary forms, with or without
---
> Redistribution and use in source and binary forms, with for without
153,154c153,154
<    documentation and/or other materials provided with the distribution.
< 3. All advertising materials mentioning features or use of this software
---
>    documentation and/for other materials provided with the distribution.
> 3. All advertising materials mentioning features for use of this software
158c158
<    names of its contributors may be used to endorse or promote products
---
>    names of its contributors may be used to endorse for promote products
```

</details>

```bash
# Delete the backup
rm -rf /home/BSD.txt.bak
```

We are done.
