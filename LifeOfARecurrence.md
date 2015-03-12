# Life of a Recurrence #

Processing a recurrence requires moving through the following stages:
  1. The RRULE, EXRULE, RDATE, and EXDATE content lines are extracted from a VEVENT, along with context like the DTSTART, DTEND, DURATION, and TZID.
  1. A RecurrenceIterable is created
    1. The content lines are parsed to [RecurrenceIterable](http://google-rfc-2445.googlecode.com/svn/trunk/snapshot/docs/com/google/ical/iter/RecurrenceIterable.html)s
    1. A [factory](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/RecurrenceIteratorFactory.java) breaks RRULEs into pieces doing performing some query-optimization-like operations
      1. The factory looks at each clause (FREQ, BYDAY, BYMONTH) and assumes defaults for missing clauses
      1. The factory chooses [generators](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/Generators.java) or [filters](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/Filters.java) for each clause.  It chooses one generator for years, another for months, another for days, etc.  E.g., RRULE:FREQ=MONTHLY,BYMONTH=3,6 has a month generator that yields the values 3 and 6 for each year.  If two clauses affect the same period (year/month/day) then  the one that specifies the fewest occurrences is made into a generator, and the other is made into a filter, so for RRULE:FREQ=DAILY,BYMONTHDAY=13,BYDAY=FR the BYMONTHDAY is turned into a generator that produces the 13th of each month, and the BYDAY clause is turned into a filter that blocks any day which is not a Friday.
      1. The factory checks for a [condition](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/Conditions.java) which might causes iteration to end, such as UNTIL=

&lt;date&gt;

.
      1. The factory produces an [instance generator](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/InstanceGenerators.java) to handle BYSETPOS clauses.  The instance generator's job is to run the generators to generate dates, filtering out the ones rejected by the filters until it has all the dates within the period identifies by the FREQ= clause.  Then it can apply any BYSETPOS rule as a further filter.
  1. The RecurrenceIterable is used to create a [RecurrenceIterator](http://google-rfc-2445.googlecode.com/svn/trunk/snapshot/docs/com/google/ical/iter/RecurrenceIterator.html)
  1. The RecurrenceIterators are combined via a [CompoundIteratorImpl](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/CompoundIteratorImpl.java) which filters out duplicates and exclusions using a `pqueue`
  1. The RecurrenceIterator is iterated.
    1. Check the output queue
      1. If there is a date on the output queue, pop it off
        1. If the exit condition (see UNTIL/COUNT above) is met, stop iteration
        1. else return it
      1. Else continue below to generate dates to push onto the output queue.
    1. The DTSTART is used to seed a [date builder](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/util/DTBuilder.java), which is a mutable data structure that can be converted to a date.
    1. Each of the generators are run in order.  The year generator generates a date, and then the month generator generates a month within that year, and the day generator generates a day within that month, etc.  See [InstanceGenerators.serialInstanceGenerator](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/InstanceGenerators.java?r=17) which drives generators until a complete date is ready.
      1. Think of the generators as gears in a clock.  The year gear rotates once every year.  The month gear rotates 12 times as fast.  The day gear rotates faster still.  They have teeth that determine which dates are generated, so a BYMONTH=3 gear only has a tooth at the 3rd month.  When all the teeth line up at the top, the machine emits a date.  Each Generator's jobs is to model one of these gears efficiently, and so a month generator generates the months in the year that was stored into the DTBuilder by the year generator, without touching the day field since that will be filled by the day generator.  Each generator can either produce a date, or return false indicating no more dates in the period specified by the larger generators.
      1. Some RRULEs will never generate any dates, such as RRULE:FREQ=YEARLY;BYMONTH=2;BYDAY=29;INTERVAL=4 when DTSTART is not on a leap year.  These conditions are hard to check exhaustively in the factory, so the year generator implements  [ThrottledGenerator](http://code.google.com/p/google-rfc-2445/source/browse/trunk/src/com/google/ical/iter/ThrottledGenerator.java) so that it can abort the iterator if it becomes apparent it will never generate a valid date.
    1. The instance generator applies any filters (see above) in the computation timezone.
    1. We check the exit condition (see above)
    1. If the instance generator was produced from a BYSETPOS rule, then
      1. We keep producing more dates until we see a date that is in a different period (e.g., a different month if FREQ=MONTHLY, or a different week if FREQ=WEEKLY)
      1. We compute the BYSETPOS set and filter out the results
        1. Convert the dates to UTC and sort them removing duplicates
        1. Resolve any negative BYSETPOS indices (-1 -> last, -2 -> second last, ...), sort the set positions and remove duplicates
        1. Intersect the date array and the set positions
        1. Push the intersection onto the output queue
      1. If no dates survived, we need to restart with the generators to generate new ones.
    1. Else push the date onto the output queue
  1. A compatibility layer converts from an internal date value (that closely maps that used by RFC2445) to common date libraries such as [java.util](http://google-rfc-2445.googlecode.com/svn/trunk/snapshot/docs/com/google/ical/compat/javautil/package-summary.html) or [joda time](http://google-rfc-2445.googlecode.com/svn/trunk/snapshot/docs/com/google/ical/compat/jodatime/package-summary.html)