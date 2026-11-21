# Recurring care and calendar edge cases

[Back to the engineering case study](../engineering.md)

## Problem and decision

A recurrence such as “every month” is different from “every 30 days.” Reminders must follow the owner's chosen local date and time, while handling shorter months and previously completed occurrences.

The Android recurrence helper aligns candidate dates to a start-date anchor. Day/week rules use calendar-day differences. Month rules rebuild each candidate from the original anchor and clamp its day to that month's length, preserving the original day for later months. For example, a monthly rule anchored on January 31 can use February's last day and then March 31.

## Actual source excerpt

The alignment helper from `android/app/src/main/kotlin/com/melisgedik/petcrew/ReminderRecurrence.kt`:

```kotlin
    private fun align(rule: ReminderRule, value: String, zone: TimeZone): Calendar {
        val anchor = day(rule.startDate, zone)
        val cursor = day(value, zone)
        if (rule.repeatEvery == 0) return cursor
        if (rule.repeatUnit != "months") {
            val utc = TimeZone.getTimeZone("UTC")
            val days = ((day(value, utc).timeInMillis - day(rule.startDate, utc).timeInMillis) / 86400000L).toInt()
            val step = rule.repeatEvery * if (rule.repeatUnit == "weeks") 7 else 1
            val rest = days % step
            if (rest != 0) cursor.add(Calendar.DAY_OF_MONTH, step - rest)
            return cursor
        }
        val months = (cursor.get(Calendar.YEAR)-anchor.get(Calendar.YEAR))*12 + cursor.get(Calendar.MONTH)-anchor.get(Calendar.MONTH)
        var index = maxOf(0,months / rule.repeatEvery)
        fun candidate(): Calendar = (anchor.clone() as Calendar).apply {
            val originalDay = anchor.get(Calendar.DAY_OF_MONTH)
            set(Calendar.DAY_OF_MONTH, 1); add(Calendar.MONTH, index * rule.repeatEvery)
            set(Calendar.DAY_OF_MONTH, minOf(originalDay,getActualMaximum(Calendar.DAY_OF_MONTH)))
        }
        var result = candidate()
        if (date(result) < value) { index++; result = candidate() }
        return result
    }
```

`ReminderRule`, date helpers, schedule validation, occurrence exclusions and Android alarm delivery are intentionally omitted. This is an excerpt for code review, not a standalone scheduling library or buildable app.

## Validation and limits

The complete project includes native recurrence tests and Flutter tests for recurring care, history and notification routing. Notifications depend on Android permissions and device restrictions; exact delivery is not guaranteed. Schedules are supplied by the owner or vet, not inferred as medical advice.
