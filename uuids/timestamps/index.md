---
layout: default
title: Timestamps
in_section: uuids
---

# Timestamps

RON serializes timestamps of variable precision using calendar-aware `MMDHmSssnn` format:

- Ten Base 64×64 chars encode months-since-epoch, days, hours, minutes, seconds, milliseconds and an arbitrary 12-bit sequence number.
- Swarm epoch starts at January 1st, 2010 00:00:00 UTC (Unix epoch plus 1262304000000 ms). This means month 0 would be Jan 2010, month 12 would be Jan 2011, month 24 would be Jan 2012 and so on.
- All timestamps use UTC time zone and Gregorian calendar.
- We do not use Unix time (number of milliseconds since 1970) because Swarm timestamps are [hybrid](https://cse.buffalo.edu/tech-reports/2014-04.pdf) (logical, calendar-aware). Human-understable timestamps are more important than easy calculation of time intervals or a bit of extra precision.
- You can generate 4,096,000 distinct timestamps for each second.
- When appropriate (e.g. it doesn’t affect sequentiality), timestamps might be shortened by zeroing the tail (drop sequence number, then milliseconds, etc). E.g. sequence number only makes sense if you generate more than one timestamp in the same *millisecond*.
- `~` is a special timestamp value, never (+infinity).

Examples:

- `1CQAneD` = `1CQAneD000` = `1C` (76th, Jan 2010-based) month, `Q` (26th, but 0-based) day, `A` (10) hour, `n` (50) minutes, `e` (41) seconds and `D0` (832) milliseconds, meaning May 27, 2016 10:50:41.832
- `1CQAn` = `1CQAn00000` May 27, 2016 10:50:00.000