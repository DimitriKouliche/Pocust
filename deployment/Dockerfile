ARG PYTHON_VERSION
FROM eu.gcr.io/ma-backbone/python/${PYTHON_VERSION}-slim:latest as python-base

COPY Pipfile Pipfile.lock ./

COPY --chown=${USER_UID}:${USER_GID} ./data ./data
COPY --chown=${USER_UID}:${USER_GID} ./stress_tests ./stress_tests

###############################################
## Target for test image
###############################################
FROM python-base as test

ARG APP_VERSION
ENV APP_VERSION=${APP_VERSION}

RUN pipenv install --deploy --dev

COPY --chown=${USER_UID}:${USER_GID} .flake8 .isort.cfg ./

###############################################
## Target for production image
###############################################
FROM python-base as prod

RUN pipenv install --deploy

CMD ["locust"]
