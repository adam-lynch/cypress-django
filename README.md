# cypress-django

Python and Node.js package providing support for Cypress and Django integration.

## Python: Cypress DB

Issue commands to operate on the project's Cypress test database.

Expects `settings/cypress.py` to exist for the Django settings.
Expects `cypress/db/setup_test_data.py` to exist to house the functions for loading
in test data.

This script can be used in Cypress tests to load data into the test database,
as well as be run on the command line as a shortcut for various operations on the
test database.

It caches the last loaded test data function and exits early if the same function
is loaded again - for tests that do not alter the database, it is not necessary to
reload the data, so we can save time here.

Implemented as a standalone script rather than a management command to avoid the
overhead of `manage.py` when exiting early, since we want tests to be as fast as
possible.

### Installation
Either install directly:

```
pip install git+https://github.com/QuickRelease/cypress-django.git
```

or add to `requirements.txt`:

```
-e git+https://github.com/QuickRelease/cypress-django.git#egg=cypress-django
```

### Usage

```
python -m cypress_db [-h] [--init] [--flush] [--clearcache] [func]
```

positional arguments:
```
  func          Setup test data function to run
```

optional arguments:
```
  -h, --help    show this help message and exit
  --init        Initialise the database (run `migrate` and `createcachetable`)
  --flush       Clear all data (run `flush`)
  --clearcache  Delete the test data cache (use when a test will modify the database)
```

## Node.js: Cypress Commands

Provides a `login` helper function:
- `cy.login()` will programmatically login using the `USERNAME` and `PASSWORD` environment
variables defined in `cypress.json` (or with `CYPRESS_` prefix if defined elsewhere)
- Recommended way to use is to put `cy.login()` in `beforeEach`
- If it is necessary to login as a different user, for example to test behaviour for users with
limited permissions, simply provide the appropriate username and password as arguments

Provides a `setupDB` helper function that expects the python package in this repo to be
installed:
- `cy.setupDB` will flush the test database and load new data according to the
function `setupFunc`
- If the test will write to the database, `mutable` should be set to `true`
- Otherwise set `mutable` to `false` to allow early exit from the script if no fixture loading
is necessary (this means repeated test runs with such tests will be significantly faster)
- The `setupFunc` argument should be the name of a function living in `cypress/db/setup_test_data.py`
which loads whatever data necessary into the test database - this is similar to a
`TestCase.setUpTestData` method 

### Installation

With `npm`:
```
npm install git+https://github.com/QuickRelease/cypress-django.git --save-dev
```

and include this line in `cypress/support/index.js` or `cypress/support/commands.js`:
```
import 'cypress-django/commands'
```
