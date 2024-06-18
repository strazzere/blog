---
title: '"Brutal" Google coding humor...'
tags:
  - android
  - AOSP
  - comment humor
  - google
  - humor
  - javadoc
url: brutal-google-coding-humor.html
id: 325
categories:
  - android
date: 2010-03-22 14:08:36
---

Today I was skimming over a few pieces of the AOSP and found the following comment to be hilarious:

> Determine whether it is a good time to kill, crash, or otherwise plunder the current situation for the overall long-term benefit of the world.

Below is the full snippet from the file, `Watchdog.java`.
```
    /**
     * Load the current Gservices settings for when
     * {@link #shouldWeBeBrutalLocked} will allow the brutality to happen.
     * Must not be called with the lock held.
     */
    void retrieveBrutalityAmount() {
        mMinScreenOff = (mReqMinScreenOff >= 0 ? mReqMinScreenOff
                : Settings.Gservices.getInt(
                mResolver, Settings.Gservices.MEMCHECK_MIN_SCREEN_OFF,
                MEMCHECK_DEFAULT_MIN_SCREEN_OFF)) * 1000;
        mMinAlarm = (mReqMinNextAlarm >= 0 ? mReqMinNextAlarm
                : Settings.Gservices.getInt(
                mResolver, Settings.Gservices.MEMCHECK_MIN_ALARM,
                MEMCHECK_DEFAULT_MIN_ALARM)) * 1000;
    }

    /**
     * Determine whether it is a good time to kill, crash, or otherwise
     * plunder the current situation for the overall long-term benefit of
     * the world.
     *
     * @param curTime The current system time.
     * @return Returns null if this is a good time, else a String with the
     * text of why it is not a good time.
     */
    String shouldWeBeBrutalLocked(long curTime) {
        if (mBattery == null || !mBattery.isPowered()) {
            return "battery";
        }

        if (mMinScreenOff >= 0 && (mPower == null ||
                mPower.timeSinceScreenOn() < mMinScreenOff)) {
            return "screen";
        }

        if (mMinAlarm >= 0 && (mAlarm == null ||
                mAlarm.timeToNextAlarm() < mMinAlarm)) {
            return "alarm";
        }

        return null;
    }
```
Just some mild humor to lighten up the day is always nice :)