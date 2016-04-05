# jasmine-sentry-reporter

Sentry reporting for failed expectations in Jasmine test suites.


## Requirements

The reporter supports Node v4.x.x with the following peer dependencies:

- [raven](https://www.npmjs.com/package/raven) v0.10.x
- [jasmine-core](https://www.npmjs.com/package/jasmine-core) v2.x.x


## Installation

```
$ npm install jasmine-sentry-reporter
```


## Setting up the reporter

Reporting to Sentry is done with a Raven client. See
[raven's documentation](https://docs.getsentry.com/hosted/clients/node/#configuring-the-client)
for more information about configuring the client.

The Raven client is the only parameter needed for constructing a
`JasmineSentryReporter`.

```js
const JasmineSentryReporter = require('jasmine-sentry-reporter');
const raven = require('raven');

const ravenClient = new raven.Client(process.env.SENTRY_DSN);

jasmine.getEnv().addReporter(new JasmineSentryReporter(ravenClient));
```

## What is reported to Sentry

The reporter will capture the exception that caused the Jasmine expectation to
fail. This error and its stack trace are reported to Sentry.

In addition, the issue will be tagged with `testCaseName` containing the full
name of the failed expectation.


### Extending the context

If you wish to add new information to the issues reported, use
[Raven's global context adding](https://docs.getsentry.com/hosted/clients/node/usage/#raven-node-additional-context). You can for example set up CI environment variables to all reported issues:

```js
ravenClient.setTagsContext({
  branch: process.env.CIRCLE_BRANCH,
  buildNum: process.env.CIRCLE_BUILD_NUM,
  commit: process.env.CIRCLE_SHA1,
  containerIndex: process.env.CIRCLE_NODE_INDEX
});
```

## Why report failing tests to Sentry?

Our organization came across with the unfortunate situation of an unstable E2E
test suite. We use Protractor and Jasmine to run our E2E tests on CircleCI
containers. Knowing which tests are unstable and how the situation evolves is
difficult through the CI's user interface.

Sentry aggregates event information well, so seeing the most common issues is
easy. Tagging enables us to filter events based on for example Git branch,
browser and test case. This allows us to distinguish between real failures and
instability.
