---
layout: post
title: Use "tmutil" to Restore files from Time Machine
categories: ["System Administration"]
---

Time Machine\'s GUI is great for your applications\' documents; but,
when you want to restore files that require superuser privileges, you
need the extra control you get from OS X\'s `tmutil` command.

`sudo tmutil listbackups` shows you which snapshots are available to
restore from:

    ...
    /Volumes/.../2014-08-30-014922
    /Volumes/.../2014-08-31-002302
    /Volumes/.../2014-09-01-022453
    /Volumes/.../2014-09-02-015445
    /Volumes/.../2014-09-03-020003
    /Volumes/.../2014-09-04-002614
    /Volumes/.../2014-09-05-023820
    /Volumes/.../2014-09-06-020504
    /Volumes/.../2014-09-07-014641
    /Volumes/.../2014-09-08-001751
    /Volumes/.../2014-09-09-003142
    /Volumes/.../2014-09-10-005409
    /Volumes/.../2014-09-10-094443
    /Volumes/.../2014-09-10-115715
    /Volumes/.../2014-09-10-141342
    /Volumes/.../2014-09-10-162715
    /Volumes/.../2014-09-10-183949
    /Volumes/.../2014-09-10-205256
    /Volumes/.../2014-09-10-230350
    /Volumes/.../2014-09-11-011645
    /Volumes/.../2014-09-11-032854
    /Volumes/.../2014-09-11-054003
    /Volumes/.../2014-09-11-075156

Then you can use `grep` and `mdfind` etc. to locate the files you want
in the particular snapshot you want.

Finally, you use `tmutil restore`:

    tmutil help restore
