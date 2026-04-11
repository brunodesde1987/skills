# Numbers, Dates & Formatting

Modern patterns for math precision, date/time handling, and formatting.

---

## Math.sumPrecise (ES2026)

Eliminates floating-point accumulation errors when summing arrays of
numbers with vastly different magnitudes. Uses a specialized algorithm
(compensated summation) that avoids precision loss in intermediate results.

```js
// ❌ OLD — precision lost with varying magnitudes
[1e20, 0.1, -1e20].reduce((a, b) => a + b, 0);  // 0 ← WRONG

// ❌ OLD — custom Kahan summation
function kahanSum(arr) {
  let sum = 0, c = 0;
  for (const val of arr) {
    const y = val - c;
    const t = sum + y;
    c = (t - sum) - y;
    sum = t;
  }
  return sum;
}

// ✅ NEW — mathematically precise, single call
Math.sumPrecise([1e20, 0.1, -1e20]);  // 0.1 ← CORRECT
```

**When to use:** Financial calculations, scientific computing, statistical
aggregation — anywhere you're summing numbers that might span many orders
of magnitude. Also useful as a simple replacement for any `.reduce((a,b) => a+b, 0)`
where precision matters.

**Support:** Firefox 137+; Chrome and Safari not yet shipped.

---

## Temporal (Stage 4 → ES2027)

The single biggest change to JavaScript's standard library. Temporal
replaces the broken `Date` object with purpose-built immutable types that
handle timezones, calendars, and duration arithmetic correctly.

**Current status:** Reached Stage 4 at the March 2026 TC39 meeting.
Expected in the ES2027 spec. Browser support is actively shipping:
Firefox 139+, Chrome 144+. Safari in development. Polyfill available
at `@js-temporal/polyfill`.

### Why Date is broken

The `Date` object has well-known problems: mutable (methods modify in
place), 0-indexed months (January = 0), no timezone support beyond local
and UTC, no duration type, no calendar support, and string parsing is
inconsistent across engines.

### Temporal's type system

| Type | What it represents | Example |
|------|-------------------|---------|
| `Temporal.Instant` | Exact moment in time (UTC) | A timestamp |
| `Temporal.ZonedDateTime` | Moment + timezone + calendar | "Meeting at 3pm ET" |
| `Temporal.PlainDate` | Calendar date (no time, no tz) | "April 3, 2025" |
| `Temporal.PlainTime` | Wall-clock time (no date, no tz) | "14:30:00" |
| `Temporal.PlainDateTime` | Date + time (no timezone) | Form input |
| `Temporal.PlainYearMonth` | Year + month only | "March 2025" |
| `Temporal.PlainMonthDay` | Month + day only | "Birthday: Dec 25" |
| `Temporal.Duration` | A length of time | "2 hours 30 minutes" |

### Basic usage

```js
// ❌ OLD — mutable, 0-indexed months, confusing
const d = new Date(2025, 0, 1);  // January (0 = January!)
d.setMonth(11);                  // mutates in place

// ✅ NEW — immutable, explicit, correct
const date = Temporal.PlainDate.from('2025-04-03');
const nextWeek = date.add({ days: 7 });     // returns NEW object
// date is still "2025-04-03"

// Current time
const now = Temporal.Now.zonedDateTimeISO();      // full timezone-aware now
const today = Temporal.Now.plainDateISO();         // just today's date
const instant = Temporal.Now.instant();            // exact UTC moment
```

### Timezone handling

```js
// ❌ OLD — no real timezone support
// Date only knows local time and UTC — no way to work with arbitrary timezones

// ✅ NEW — first-class IANA timezone support
const meeting = Temporal.ZonedDateTime.from({
  year: 2025, month: 12, day: 15, hour: 10,
  timeZone: 'America/New_York'
});
const inTokyo = meeting.withTimeZone('Asia/Tokyo');
// Automatically handles DST transitions

// Converting between timezones
const departure = Temporal.ZonedDateTime.from('2025-06-15T08:00[America/Los_Angeles]');
const arrival = departure.add({ hours: 11 }).withTimeZone('Europe/London');
```

### Duration arithmetic

```js
// ❌ OLD — manual millisecond math, error-prone
const twoHoursMs = 2 * 60 * 60 * 1000;
const future = new Date(Date.now() + twoHoursMs);

// ✅ NEW — explicit duration type
const duration = Temporal.Duration.from({ hours: 2, minutes: 30 });
const meeting = Temporal.PlainTime.from('14:00');
const end = meeting.add(duration); // "16:30:00"

// Comparing durations
const d1 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d2 = Temporal.Duration.from({ minutes: 100 });
Temporal.Duration.compare(d1, d2); // -1 (d1 is shorter: 90 min < 100 min)
```

### Date comparison and difference

```js
const start = Temporal.PlainDate.from('2025-01-15');
const end = Temporal.PlainDate.from('2025-04-03');

// Difference with specified largest unit
const diff = start.until(end, { largestUnit: 'month' });
// { months: 2, days: 19 }

// Comparison
Temporal.PlainDate.compare(start, end); // -1 (start is earlier)
```

### Important: do not suggest Temporal without noting its status

When suggesting Temporal, always mention:
1. It requires a polyfill for production use today (`@js-temporal/polyfill`)
2. Native browser support is still rolling out (Firefox 139+, Chrome 144+)
3. The polyfill adds bundle size (~40KB minified)
4. For simple date formatting, `Intl.DateTimeFormat` may be sufficient
