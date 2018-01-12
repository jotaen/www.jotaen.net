+++
draft = true
title = "Open heart surgery"
subtitle = "Successful data migrations during full operation"
date = "2018-01-08"
tags = ["database"]
image = "/posts/2018-01-08-data-migrations/engine.jpg"
id = "asdf9"
url = "asdf9/data-migrations"
aliases = ["asdf9"]
+++

Modern web applications are expected to run 24/7 without noticeable downtime. While code changes can be pushed out almost instantaneously due to modern continuous delivery pipelines, database migrations are a delicate and rather expensive process.

Freezing the database (and therefore the application as a whole) is not viable because of business demands, even though it would make the lives of software developers a lot simpler. Minor database migrations also don’t justify to disrupt the development activity – instead, they should be seemless, safe and gradual.

This blog post outlines an exemplary procedure. It should be applicable for the smaller kinds of migrations that developers are facing during their daily work.

# Breaking new ground

I want to guide you to the procedure by means of an example. One word about technology upfront: the sample code is written in NodeJS and we are using MongoDB as database (which is a document store). In the end, it shouldn’t really matter though what kind of database you are using and whether you employ a dynamically or statically typed language. The underlying ideas are the same and the concept would be even similar if you migrated a public HTTP API.

## Initial situation

Let’s say we have a module that manages employees in an application. All user data is stored in a MongoDB, which the module has full ownership of. (This means there exists no other piece of code that has direct access to the database.)

The database documents which represent the employees look like this in the collection:

```json
{
    _id: ObjectId("7abe787zabce78a12"),
    name: "John Doe",
    employedSince: 1979,
    workplace: "Buenos Aires"
}
```

Requirements change and the application shall now also support employees who work from multiple locations:

```json
{
    _id: ObjectId("7abe787zabce78a12"),
    name: "John Doe",
    employedSince: 1979,
    locations: ["Buenos Aires", "Singapore", "Berlin"]
}
```

So, effectively we want two things here: rename the field and convert the value to the new format.

## 1. Support symmetric writes and reads

We introduce the new field in the database layer of the application. For **read operations** we derive the migration status from the presence of the new field. The new field is favoured but the code falls back to the old one:

```js
employees.read = (employeeId) => {
    return mongodb.findOne({
        _id: ObjectId(employeeId)
    }).then((employee) => {
        const isMigrated = Array.isArray(employee.locations);
        return {
            name: employee.name,
            employedSince: employee.employedSince,
            workplace: isMigrated ? employee.locations[0] : employee.workplace,
            locations: isMigrated ? employee.locations || [employee.workplace]
        };
    });
};
```

For **write operations** (for instance creation) we introduce an explicit parameter that the migrated client code would set to false. We again fall back to `workplace`. A content mismatch of the two properties can be intercepted to be on the safe side with consitency.

```js
employees.create = (employee, isMigrated) => {
    if (isMigrated) {
        if (employee.locations.length > 1) {
            throw new Error("There can only be 1 location max.");
        }
        if (employee.workplace && employee.workplace !== employee.locations[0]) {
            throw new Error("Old and new location value must be equal.")
        }
    }
    return mongodb.insert({
        name: employee.name,
        employedSince: employee.employedSince,
        workplace: isMigrated ? employee.locations[0] : employee.workplace,
        locations: isMigrated ? employee.locations : [employee.workplace]
    });
};
```

Note that `locations` is introduced with a constraint that makes it fully backwards compatible. We also do not omit the old field right away, we rather keep it up to date and consistent. This not only reduces complexity while the migration is in progress, it is also crucial in the event of a deployment rollback, which means that a previous application version would come into effect again. The problem is here that documents with the new field could have already been written in the meantime, which the old code version is unable to gracefully deal with. Be it likely or not, but by ignoring this you literally slam the door behind you upon your next deployment – the way back is locked.

## 2. Migrate all client code

With the first step the new property has become available to the application and the old one is deprecated. We can take all the necessary time to refactor the places where employee objects are used to the new format. However, there is of course still the constraint that only one value is supported in the array!

If it is possible to safely refactor the entire client code in one atomic transition along with the changes to the database layer, you can simplify the implementation by omitting the migration flag in the database service API.

## 3. Stop persisting the deprecated property

Once the period of grace for a potential rollback has elapsed we can discontinue to write the old property into the database. In this example, we are effectively only removing a single line of code:

```js
// ...
    return mongodb.insert({
        name: employee.name,
        employedSince: employee.employedSince,
        locations: employee.locations || [employee.workplace]
    });
// ...
```

## 4. Migrate the data

With the previous step, the values of all old properties are effectively frozen. That allows us to run over all the user documents in the collection and persist the conversion that we did on the fly in our db module above:

```js
let migrationCount = 0;

db.find({}).forEach((employee) => {
    const isMigrated = Array.isArray(employee.locations);
    mongodb.update({ _id: employee._id }, {
        name: employee.name,
        employedSince: employee.employedSince,
        locations: isMigrated ? employee.locations : [employee.workplace]
    }).then(() => migrationCount++);
});

console.log('Migrated: ' + migrationCount);
```

Generally you should keep in mind to write your migration algorithm in an idempotent way, so that you can safely run it multiple times. (Imagine the database connection failed halfway in between and you’d need to start over.)

If you want to have an extra safety net, you don’t have to drop the old field right away. You could also keep it in the database for some time and then clean it up later.

## 5. Drop all support for the deprecated property

Now that the data has been fully migrated we can drop the support of the deprecated property to the outside world.

```js
employees.create = (employee) => {
    return mongodb.insert({
        name: employee.name,
        employedSince: employee.employedSince,
        locations: employee.locations
    });
};

employees.read = (employeeId) => {
    return mongodb.findOne({
        _id: ObjectId(employeeId)
    }).then((employee) => {
        return {
            name: employee.name,
            employedSince: employee.employedSince,
            locations: employee.locations
        };
    });
};
```

Once this has happened the constraint for the new property is obsolete and the consumers can take full advantage of the new field supporting a superset of values. The migration is thereby successfully completed.

# Basic recipe

Essentially, this is the basic recipe for a migration from `old` to `new`:

1. Introduce `new`:
  - Make it fully compatible through constraints.
  - Always write both the old and new field.
  - For read operations fall back to `old` if `new` isn’t set yet.
2. Deprecate `old` and migrate all consumers of the API.
3. Shift sole write sovereignity to `new`. Read operations continue to fall back to `old`.
4. Run a migration to copy all values from `old` to `new`.
5. Remove `old` altogether from the code.

In practice the migration plan probably varies here and there. Depending on the circumstances you may take shortcuts, but it is also very well possible that a migration cannot be accomplished within one process at all. There are also slight technical differences depending on the kind of database that is being used.

In the end, the important thing is to come up with a plan upfront that describes all steps as concise and explicit as possible. The procedure can be intimidating, especially for developers who are not so experienced with it, which makes it crucial for everyone to know what the current status is and what comes next.

# Asymmetric migration

The first example showed a symmetric migration – during the transition period we enforced a constraint that allowed us to compute the new property from the old one and vice versa. This, however, is not always possible.

- Migrations can be destructive, if you shift over to a format that only allows for a subset of the previously available value range.
- On the other hand, even if you migrated to a format that supports a superset of possible values, it can still happen that the new format calls for information that was just not existing previously.

In order to illustrate both cases, let’s say we wanted to migrate the date of employment from only the year (stored as integer value) to a fully qualified date object. In addition to that you want to migrate backwards from multiple `locations` to only a singular `workplace`. The desired end result would look like this:

```json
{
    _id: ObjectId("7abe787zabce78a12"),
    name: "John Doe",
    employmentDate: Date("1979-08-15"),
    workplace: "Singapore"
}
```

These kinds of migrations are non-trivial and cannot be handled by the database layer alone. While it’s obvious that we can do *something* about the location information, there is no reasonable way to compute the full date out of nothing. This leaves us with basically three options:

## Removal of data

If the value is optional for the application and not critical for the business, we introduce the new fields additionally and initialise them with `null`:

```json
{
    _id: ObjectId("7abe787zabce78a12"),
    name: "John Doe",
    employmentDate: null,
    employedSince: 1979,
    workplace: null,
    locations: ["Buenos Aires", "Singapore", "Berlin"]
}
```

The application would stop providing write support for the old fields and encourage its users to enter the new information. At some point the deprecated fields will eventually be frozen and made read-only in the database layer. They can then either be dropped or kept around for historical reference. Note, that especially in document stores the latter has a potential pitfall though: if someone else reinvents the old fields at some point in the future without knowing or checking that they had already existed in the past, they might find themselves badly surprised to retrieve “random” values for their “new” field. One possible workaround is to rename the property to something obvious like `archivedField_workplace`.

## Best guess

We could try to calculate a best guess for the new data format:

- For the employment date we would have to assume an arbitrary month and day, e.g. `Date("1979-01-01")`. Not really a feaseble option here, though.
- For `locations` there are two ways. We can either choose one of the values from the array or we can join the array together to one, comma-separated string. Both are not immaculate though: the first means losing data, whereas the second wouldn’t be a singular value in the logical sense.

## Locking

As the very last resort, if backwards compatibility is terminated and the data can neither be guessed nor made optional, the document must be locked until the end user has manually updated their data.

```js
employees.read = (employeeId) => {
    const employee = mongodb.findOne({
        _id: ObjectId(employeeId)
    }).then((employee) => {
        if (!employee.employmentDate) {
            throw new Error("Document invalid: employmentDate missing.")
        }
        if (!employee.workplace) {
            throw new Error("Document invalid: workplace missing.")
        }
        return {
            name: employee.name,
            employedSince: employee.employedSince,
            locations: employee.locations
        };
    });
};
```