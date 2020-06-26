![CI](https://github.com/voidus/coronaSignIn/workflows/CI/badge.svg)
# Simple Sign-In system to comply with Hamburg, Germany corona regulations

Open questions:
- Database Backups

ToDos:
- Make it look nicer
- Some more features from the list with the link I don't have

## Dev Setup

The application requires Python version 3.8.
With `direnv` installed, you only need to install pipenv and then you're set.

Otherwise, you need to install `pipenv` first. Then, install the dependencies
using `pipenv install`.
You can run the project in development mode using
```
env \
    FLASK_ENV=development \
    CORONA_SIGN_IN_SECRET_KEY="insecure secret key" \
    CORONA_SIGN_IN_DATABASE_URI="sqlite:///${PWD}/db.sqlite" \
    pipenv run flask run
```

If you have direnv installed (highly recommended), this will work automatically,
and you can start the dev server using `flask run`

### Tests

Run the tests using `pipenv run pytest`. You can also auto-rerun tests while
you're changing code with `pipenv run ./tdd.sh`. It will only re-run tests
affected by your changes, so it should be a pretty good feedback loop.

Note: You need to have chromedriver installed for the selenium tests. If that is
not the case, you can run all non-selenium tests using `pytest -m 'not slow'`

### Style

Use (black)[https://github.com/ambv/black] to format your code. It will be
installed as part of the dev dependencies.

### Pre-Commit

To automatically run black and check for a few other common issues on commit,
run `pre-commit install`. It is installed as part of the dev requirements and
will set everything up. You only ever need to run it once.

## Configuration

Copy the `config.py.example` to `config.py` and adjust the relevant values.

## Database

Run `flask db upgrade`.


## Deployment

### Building the docker image

`podman build -t corona-sign-in .`

### Migrating the database

NOTE: if you want to use something besides postgres, additional dependencies
might need to be installed

`podman run -e CORONA_SIGN_IN_DATABASE_URI=postgresql://user:pw@host/database
corona-sign-in flask db upgrade`

### Running the application manually

1. Build the image
    `podman build -t corona-sign-in .`
2. Create the pod
    `podman pod create --name corona-sign-in --publish 8080:8080`
3. Run the database
    `podman container run -d --name corona-sign-in-db --pod corona-sign-in -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=corona-sign-in -e POSTGRES_DB=corona-sign-in postgres`
4. Migrate the database
    `podman container run --pod corona-sign-in -e CORONA_SIGN_IN_DATABASE_URI="postgresql://corona-sign-in:pass@localhost/corona-sign-in" -e CORONA_SIGN_IN_SECRET_KEY="irrelevant" corona-sign-in flask db upgrade`
5. Run the application
    `podman container run -d --pod corona-sign-in -e CORONA_SIGN_IN_DATABASE_URI="postgresql://corona-sign-in:pass@localhost/corona-sign-in" -e CORONA_SIGN_IN_SECRET_KEY="irrelevant" corona-sign-in`

### Running the application using the kube file

1. Build the image
    `podman build -t corona-sign-in .`
2. Load the kube file
    `podman play kube kube-dev.yml`
3. Migrate the database
    `podman exec corona-sign-in-app flask db upgrade`

#### Get a database shell

`podman container run -ti --rm --pod corona-sign-in postgres -h localhost -U corona-sign-in corona-sign-in`
