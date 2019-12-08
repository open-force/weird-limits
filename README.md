# Weird Limits (https://github.com/open-force/weird-limits/)

***Inspired by [wtfapex](https://github.com/ChuckJonas/wtfapex)***

This is a collection of Salesforce limits that are esoteric, unusual, interesting, or relatively unknown, as well as some other platform oddities.

The intent of this document is to shine a bit of light into some of the odd corners of a complicated platform, in the hope that knowing one of these wrinkles might help you avoid getting caught out by one of these.

### CPU limits

The current challenge with the 10-second CPU limit (shared across namespaces during execution contexts) is twofold:
1. It's not clear who the worst offender is: it can be a bit of a game of hot potato where the last thing to run (often a piece of Apex code) looks like it is the reason the limit was hit when in fact the vast majority of the time was consumed elsewhere. This code was just the last thing to run.
2. Tracing usage per element. It's not clear, and difficult to debug, what slices of the CPU limit were consumed by each process, flow, workflow, apex class, trigger, etc.

### Heap size

Heap size is measured at intervals, and consequently you can violate it as long as you are back under the limit by the next interval. Certainly not recommended!

https://salesforce.stackexchange.com/questions/190859/limits-getheapsize-wildly-inaccurate/190868#190868

### For Loops w/ queryMore don't work with Custom Metadata Types

There is a known bug with querying Custom Metadata Type records if you're using the for(MyRecord__mdt record : [SELECT ]) {} structure.

The results are quite unreliable. The workaround is to query for the data in its entirety, then loop over it after the query, instead of doing it all as one operation:

List<MyRecord__mdt> records = [SELECT ...];
for(MyRecord__mdt record : records) {
}

https://success.salesforce.com/issues_view?id=a1p3A0000008x3ZQAQ&title=custom-metadata-type-soql-not-giving-proper-results

### Boxcarred Lightning Actions share limits (Fixed in Winter '20)

https://releasenotes.docs.salesforce.com/en-us/winter20/release-notes/rn_lc_predictable_apex_limits.htm

### Not all SessionIds are created equal

Sessions in Lightning have reduced privileges.

https://salesforce.stackexchange.com/questions/110515/getting-session-id-in-lightning

### Lookup Relationships can be configured not to lock the parent

Master-detail relationships always lock the parent on edits to the child. For lookups, you have the ability to configure whether this happens through your choice about what happens on record delete. One choice locks, one choice does not.

> Locks only occur if lookup relationship is not configured to clear the value of this field if the lookup record is deleted.

http://resources.docs.salesforce.com/194/0/en-us/sfdc/pdf/record_locking_cheatsheet.pdf

### You can create custom Apex classes with the same names as standard Apex classes

Salesforce will totally let you create an Apex class called Test. Don't do that :)

### Spanning relationships limit

> The limit of spanning relationships per object is 15. This means that an object can only contain up to 15 different object references.

https://help.salesforce.com/articleView?id=000316969&type=1&mode=1

### Platform Events count as DML

This means if you fire a platform event before you do say, an Apex callout, you will get an error about having DML before a callout.

### FOR UPDATE releasing locks

If you use FOR UPDATE you lock the records, but if you do a callout the lock will be released without any sort of exception or message.

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/langCon_apex_locking_statements.htm

### Bulk API 2.0 can't be called from Apex Callouts because it requires the PATCH verb

The Bulk API 2.0 requires the PATCH http verb as part of calling it, but Apex Callouts don't support this verb. So you can't use the Bulk API 2.0 between two Salesforce orgs.

### Upgrading Customer Community licenses

If you change a User from a Customer Community License up to a Partner Community license, you will not be able to change them back to a Customer Community License.

### Mixed DML (Setup and non-Setup objects)

You can't write certain types of objects together in the same execution context, or if you do there are some caveats.

> You can insert a user in a transaction with other sObjects in Apex code saved using Salesforce API version 15.0 and later if UserRoleId is specified as null.

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dml_non_mix_sobjects.htm

### Can't upsert with three-argument method and a generic List<SObject>

You cannot use the three-argument Database.upsert() method with a generic List<SObject> (as opposed to doing single argument and two argument by using `upsert myList;`, for example).

You must use a typed list (such as List<Account>) or you'll get a runtime error.

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_methods_system_database.htm#apex_System_Database_upsert

### There is a concurrent long-running transaction limit in Apex

> Number of synchronous concurrent transactions for long-running transactions that last longer than 5 seconds for each org: 10
>
> If more transactions are started while the 10 long-running transactions are still running, they’re denied. HTTP callout processing time is not included when calculating this limit.

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm

### Non-selective queries on large object types are not allowed on trigger context

> By design, non-selective queries on large object types are not allowed on trigger context (when evaluating Apex triggers). This design restriction prevents the impact a non-deterministic query execution time (due to a full/partial table scan required to perform the affected SOQL query) could have on the end-user's experience.

https://help.salesforce.com/articleView?id=000333150&language=en_US&type=1&mode=1

### The Apex Metadata API doesn't support the delete operation

From Apex you can create cMDT records, you can update cMDT records, but you can't delete cMDT records.

### Hourly limit for scheduled actions

> Groups of scheduled actions that are executed or flow interviews that are resumed per hour: 1000

https://help.salesforce.com/articleView?id=process_limits.htm&type=5

### Email limits in Apex don't match the docs

> The docs and this answer are both wrong compared to current behavior. You are allowed 10x Messaging.sendEmail calls, 100x emails per email message, and unlimited (?!) emails in a single call to Messaging.sendEmail (subject to governor limits). I just tested this and sent myself 10,000 emails in a single Messaging.sendEmail call (then canceled it to avoid spamming myself).

https://salesforce.stackexchange.com/questions/71760/apex-email-limits-clarification/71761#comment388979_71761

### ConnectAPI Apex code requires @SeeAllData to test

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/connectAPI_TestingApex.htm

### You can only have one Continuation at a time from a Lightning component session

https://developer.salesforce.com/docs/component-library/documentation/lwc/lwc.apex_continuations_limits

### Process Builder cloning

> There isn’t a way to clone or create a new process off of a current Process Builder if it uses a ‘contains’ function on an encrypted field in its filterable criteria.  Had to rebuild. -Dennis Morris


## Packaging (worthy of its own sub-section)

### You can't remove Aura components from a managed package once they have been released

Not much else to say for this one, other than ugh. There are a bunch of different components that can't be removed. There may be some hope on the horizon for this one.

### Custom Metadata Type records created in a subscriber org will block uninstall of the package until they are removed

### BatchApexErrorEvent Apex handlers cannot be added to packages (but classes that fire the event can)

### packaging tests can fail which have passed in metadata deploy 

IIRC you need to wrap all mocked callouts in Test.startTest() and stopTest(), which is good practice anyway

### Post install will fail in certain operations (typically security checks) which run under with sharing

And more confusingly inherited sharing called from without sharing all of your postinstall code paths should probably remain within without sharing classes

### Push upgrades can invoke apex as part of the push upgrade, and this runs with the same permissions as post install

This and the two prior are from James M.
