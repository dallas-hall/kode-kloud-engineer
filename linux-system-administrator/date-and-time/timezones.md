# Timezones

## Task

> During the daily standup, it was pointed out that the timezone across `Nautilus Application Servers` in `Stratos Datacenter` doesn't match with that of the local datacenter's timezone, which is `America/Belize`.<br><br>Correct the mismatch.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* How many app servers in Stratos Datacenter? 3: stapp01, stapp02, and stapp03.
* How to change a timezone in Linux - https://linuxize.com/post/how-to-set-or-change-timezone-in-linux/

## Steps

```bash
# Connect to the first app server
ssh tony@stapp01

# Get root
sudo -i

# Check current Linux version, it was CentOS Stream 8
cat /etc/*rel*

# Check current timezone
timedatectl
```

```
               Local time: Sat 2023-08-19 00:00:47 UTC
           Universal time: Sat 2023-08-19 00:00:47 UTC
                 RTC time: n/a
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
```

```bash
# Change timezone as root
timedatectl set-timezone America/Belize

# Check change
timedatectl
```

```
               Local time: Fri 2023-08-18 18:01:12 CST
           Universal time: Sat 2023-08-19 00:01:12 UTC
                 RTC time: n/a
                Time zone: America/Belize (CST, -0600)
System clock synchronized: yes
              NTP service: n/a
```

```bash
# Repeat above for
ssh steve@stapp02
ssh banner@stapp03
```