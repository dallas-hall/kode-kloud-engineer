# Timezones

## Task

> During the daily standup, it was pointed out that the timezone across `Nautilus Application Servers` in `Stratos Datacenter` doesn't match with that of the local datacenter's timezone, which is `Europe/Amsterdam`.<br><br>Correct the mismatch.

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

# Check current timezone
```

```
[tony@stapp01 ~]$ timedatectl
      Local time: Tue 2022-08-02 23:02:27 UTC
  Universal time: Tue 2022-08-02 23:02:27 UTC
        RTC time: n/a
       Time zone: UTC (UTC, +0000)
     NTP enabled: n/a
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

```bash
# Change timezone as root
sudo timedatectl set-timezone Europe/Amsterdam

# Check change
```

```
[tony@stapp01 ~]$ timedatectl
      Local time: Wed 2022-08-03 01:05:03 CEST
  Universal time: Tue 2022-08-02 23:05:03 UTC
        RTC time: n/a
       Time zone: Europe/Amsterdam (CEST, +0200)
     NTP enabled: n/a
NTP synchronized: yes
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  Sun 2022-03-27 01:59:59 CET
                  Sun 2022-03-27 03:00:00 CEST
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  Sun 2022-10-30 02:59:59 CEST
                  Sun 2022-10-30 02:00:00 CET
```

```bash
# Repeat above for
ssh steve@stapp02
ssh banner@stapp03
```