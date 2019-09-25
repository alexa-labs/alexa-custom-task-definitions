# Alexa Skill Custom Tasks Repository

## Welcome!

This repository contains sample Custom Task definitions that will show you how to define your own Custom Tasks using
OpenAPI specification.

Task is description and implementation of action that a skill provides.
It is not a user facing concept and should be transparent to the user. Each individual invocation of a Task is
uniquely identified. A Task should have following characteristics:

* Input modality agnostic - A task’s definition and functionality implementation should not change with different
  input modalities. For example, a task to “schedule taxi reservation” should retain its functionality’s behavior
  irrespective of it being launched via voice, touch or other means;
* Composable - Different tasks can work together to compose new tasks via the use of Skill Connections;
* Has a beginning and an end - Each task begins either via LaunchRequest or IntentRequest and ends with the
  use of Tasks.CompleteTask directive;
* Can have user interactions - Tasks are used to provide units of experiences, thus can engage users in interactions
  such as multi-turn dialogs, rendering device screen cards etc..

There are two types of tasks: Built-in tasks and Custom tasks. Built-in tasks' definitions are authored and maintained
by Amazon, allowing multiple skills to implement the same task definition. Their names starts with `AMAZON.`. Custom
tasks are authored by individual skills, allowing developers to divide their skills into smaller functionalities.

## Create a New Custom Task

When you create a new custom task definition, you provide a path definition in OpenAPI specification format. Each task
definition describes an entry point into your skill to start a specific piece of functionality, which include the
following elements:

* Name of the task that other skill can reference to when invoking via Skill Connections;
* Input parameters;
* Response statuses;
* An optional resulting properties.

For example, to define a functionality to start a count down via voice, you might have the following:
* Name: `CountDown`
* Input parameters:
  * `lowerLimit` of type `number`.
  * `upperLimit` of type `number`.
* Response statuses:
  * `200` when the count down was successful.
  * `500` when there were some internal errors that occurred while doing the countdown.
* Resulting properties when the status is `200`:
  * `endTime` representing the skill's system time when the count down finished.

Each OpenAPI specification file contain one to many task specifications that are grouped together. By grouping
them together, they share the same versioning, search tags and license.

To create a new custom task definition:

1. Fork this repository: https://github.com/alexa-labs/alexa-custom-task-definitions
2. Create a top level directory for your own skill.
3. In the skill directory, follow below sections to create the task definition OpenAPI specification files
   , a `README.md` describing the tasks defined and a list of example valid JSON payloads.
4. Do a pull request for Alexa team to review.

Your directory should look like the following:
```
<SkillName>/
    tasks/
        <TaskName>.<TaskVersion>.json
    samples/
        <TaskName>/<TaskVersion>/
            <Sample1>.json
            <Sample2>.json
            ...
    README.md
```

## Task Definition File

The Task Definition File is a document that follows the
[OpenAPI Spec V3.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md) specification,
with some restrictions:

1. The *name* of the task is the path without the first backward slash - e.g. `/CountDown`.
2. The *input parameters* of the task is defined using the post body of the specified path above, and ONLY post is
   supported.
3. *requestBody* of the post supports application/json only;
4. *content* in response supports application/json only.

For example, if you would like to group all your tasks together, you could create a single file `tasks.{version}.json`
. When wanting to have separate versioning per task,
it's a best practice to let the file name reflect the task name and version - e.g. `CountDown.1.json`.

## Task Name and Version Requirements

The name of the task can only contain *case-sensitive alphanumerical* characters and underscores. No spaces or special
characters are allowed. As a rule of thumb, use PascalCase for the task name.

Note that the built-in tasks use the `AMAZON` namespace, so they are specified using a period, like this:
`AMAZON.ScheduleTaxiReservation`. When fully registered, custom tasks will be namespaced with the skill ID using
the same notation. For example, `amzn1.alexa.skills.0000-0000-0000-0001.CountDown`.

## Defining Input Parameters

Once you know what functionality your task will provide, you can now start defining the parameters that the task
will accept. For example, as a task to perform a count down using voice on Alexa, we would need:

* The number to count down from.
* The number to count down to.

As a best practice, input parameters follow the camelCase naming convention.
Therefore, we would have `upperLimit` and `lowerLimit` to represent the above two
parameters.

Once we have the names, we can assign a type to the parameters. Both of the
parameters are numbers, so we will use the `number` type from OpenAPI spec. For
more types, please see https://swagger.io/docs/specification/data-models/data-types/ 
.

Optionally, we can specify contraints for the parameters. For both `upperLimit`
and `lowerLimit`, we would like to limit the number within `1` and `100`.

The completed parameters definition look like the following in JSON format:

```json
{
...
"paths": {
  "/CountDown": {
    "post": {
      "requestBody": {
        "content": {
          "application/json": {
            "schema": {
              "type": "object",
              "properties": {
                "upperLimit": {
                  "type": "number",
                  "maximum": 100,
                  "minimum": 1
                },
                "lowerLimit": {
                  "type": "number",
                  "maximum": 100,
                  "minimum": 1
                }
              }
            }
          }
        }
      }
    }
  }
}
...
}
```

## Defining Response Statuses

The response statuses follow HTTP status codes and share the same semantics. For
example, to represent the different statuses for count down:

* **200** - When the count down finishes successfully.
* **400** - When the given parameters fail validation - e.g. when `lowerLimit` is
            higher than `upperLimit`.
* **500** - When the count down fails due to an internal error.

Some statuses are provided by default when a task is invoked:

* **400** - When request fails schema validation against the given task definition -
            e.g `lowerLimit` is set to 102, higher than the specified `maximum`
            constraint of 100.
* **404** - When the requested task does not exist or no providers exist for the task.
* **403** - When the requested task has denied access

## (Optional) Defining Resulting Properties

As part of the response, in addition to status codes, we can also include a payload.
For example, when the count down finishes, we could return the time at which the count
down finished:

```json
{
...
"paths": {
  "/CountDown": {
  ...
    "post": {
      ...
      "responses": {
        "200": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "endTime": {
                    "type": "string",
                    "format": "date-time"
                  }
                }
              }
            }
          }
        }
      }
      ...
    }
  ...
  }
}
...
}
```

## (Optional) Splitting Up Definitions Through References
Since we follow OpenAPI specification, references through `$ref` are also supported.
Please see `samples/SuperCountdown/CountDown.1.json` for the full example.

## Providing Sample Payloads
For certification purposes, we would need sample payloads as to validate the schemas
that you have added. For the `CountDown.1.json` task schema example above, the below
would be a valid payload:

```
{
    "upperLimit": 10,
    "lowerLimit": 1
}
```
