+++
title = "Coding J4N.IO with NodeJS"
subtitle = "From the series “Let’s build a REST service”"
date = "2016-04-04"
tags = ["project", "coding", "backend", "database"]
image = "strandkorbs-at-beach.jpg"
id = "Q6eUW"
url = "Q6eUW/coding-j4nio-with-nodejs"
aliases = ["Q6eUW"]
+++

> This blogpost is part of the series [“Let’s build a REST service”](/Toqw4/lets-build-a-rest-service), in which I build my own shortlink webservice. The source code of the project is [on Github](https://github.com/jotaen/j4n.io).

Enough of the talking – let’s roll up the sleeves. I want to point out though, that this is not a hands-on tutorial that goes into detail on specific technologies. There are hundreds of thousands of guides on how to setup an ExpressJS project and dozens on blogposts on which NPM module is the best choice with MongoDB.

I rather like to concentrate on the general setup and the basic ideas behind the project, since these thoughts remain valid beyond particular (and frequently changing) implementation details.

# Technologies

Everyone is talking about JavaScript and NodeJS these days; packages spring up like mushrooms on npmjs.org (but also drown en masse, by the way); and an outsider could be left with the impression, that the MEAN stack is the real deal nowadays for writing web applications.

Of course, that is not true. Every technology has strengths and weaknesses and it is left to the software developer to make a considerate decision on which tool to select. Just picking en-vogue technologies that everyone goes with is a mistake that can viciously come back to roost.

However, for the j4n.io project it indeed suggested itself to pick NodeJS with MongoDB – for the following reasons:

- MongoDB is a non-relational document store. Since the shortlink datasets aren’t connected with each other and do not need to be poured into a strong schema, this database type seems to be an appropriate choice.
- I selected ExpressJS as HTTP framework, because it lives up to its name: Setting up a microservice is straightforward and free of unnecessary overhead. Handling JSON data structures is built-in. And hosting and deployment is cheap and easy.

# Project structure

I grouped the source files in the [`app` directory of my project](https://github.com/jotaen/j4n.io) into the three basic categories “bootstrapping”, “business logic” and “http logic”. These are not just metaphysic abstractions layers, but the according files are indeed separated in distinct folders. The touchpoint between these three abstraction levels is kept as strictly defined and minimal as possible.

The `app` folder contains about 350 lines of code, which is a good dimension for an application of that size. Although “lines of code” is a KPI, that certainly has to be handled with care, the size of the project is a significant property here: A microservice should not only be lightweight in regards of the size of its API.

## Business logic
The business logic of the shortlink service consists of data pre-processing and persistence. These modules have a generic API – they do not know anything about HTTP or the framework they are embedded in. If I would choose a different HTTP framework or decide to write a CLI interface for accessing the database, I could reuse these modules without having to modify a single line of code. However, portability is not the primary reason for this strict demarcation: It’s rather a matter of good and clear code structure, which is achieved by defining abstraction layers precisely and keeping them strictly separate from one another.

## HTTP logic
As most HTTP frameworks, ExpressJS offers middlewares and a controller to process the request. The middlewares enclose the controller like onion skins, which the request have to pass through. In the j4n.io project the middlewares accomplish typical tasks like request validation, data sanitization or protecting specific routes with basic authentication. That way, if a request had made it all the way down to the controller, the request can be considered valid and safe.

As I described above, I consider it to be good practice to split between the logical layers: In this case the business unit is the one, that *does* something (i.e. an algorithm or a database operation), and the HTTP module is the one, that *uses* something (i.e. the business unit).

## Setup, bootstrapping
When you look into the `bootstrap` folder, you instantly smell the dust of side-effects. Here is the place for all the dirty stuff, like accessing the process environment, establishing a database connection or starting the actual app. Because of all the global state, bootstrapping is difficult to deal with and can barely be tackled by tests. For that reason, it is good to keep these things simple and manage them all in one single place. In return, the rest of the project is completely free of this: There is no other database connection to be opened and no environment variable to be read.

# Read on

> [**Part 4: Testing and QA**](/v24iU/testing-and-qa-of-j4nio) Since microservices are small in scope and pretty much well-defined, they are not so hard to test. However, there is a whole bunch of testing utility and quality assurance tools to choose from.


<!-- *[NPM]: NodeJS Package Manager -->
