#* Variables
SHELL := /usr/bin/env bash
PYTHON := python

.SUFFIXES:

.PHONY: all
all: build check-codestyle mypy test docs

.PHONY: build
build: .git/hooks/pre-commit poetry.lock

.PHONY: poetry-download
poetry-download:
	pipx install poetry
	poetry self add poetry-plugin-up


# Ensures that we run a build on first checkout, when everything has the same
# timestamp.
.first-build:
	touch .first-build


poetry.lock: pyproject.toml .first-build
	poetry lock -n --no-update
	poetry install -n
	-poetry run mypy --install-types --non-interactive ./src ./tests
	touch poetry.lock

.git:
	git init
	git remote add origin git@github.com:{{ cookiecutter.github_user }}/{{ cookiecutter.project_name }}.git

.git/hooks/pre-commit: .git poetry.lock .pre-commit-config.yaml .pre-commit-hook
	poetry run pre-commit install -f
	install -m755 .pre-commit-hook .git/hooks/pre-commit

.PHONY: update
update:
	poetry lock -n
	poetry install -n
	-poetry run mypy --install-types --non-interactive ./src ./tests

.PHONY: pre-commit-install
pre-commit-install: .git/hooks/pre-commit

.PHONY: format
format: poetry.lock
	find src/ tests/ -name '*.py' | xargs poetry run pyupgrade --exit-zero-even-if-changed --py{{ cookiecutter.python_version.replace('.', '') }}-plus
	poetry run isort --settings-path pyproject.toml ./src ./tests
	poetry run black --config pyproject.toml ./src ./tests

.PHONY: test
test: poetry.lock
	poetry run pytest -c pyproject.toml --xdoctest ./tests ./src

.PHONY: check-codestyle
check-codestyle: poetry.lock
	poetry run isort --diff --check-only --settings-path pyproject.toml ./src ./tests
	poetry run black --diff --check --config pyproject.toml ./src ./tests
	poetry run flake8 ./src ./tests

.PHONY: mypy
mypy: poetry.lock
	poetry run mypy --config-file pyproject.toml ./src ./tests

.PHONY: pre-commit
pre-commit: .git/hooks/pre-commit
	poetry run pre-commit run --all-files


.PHONY: lint pre-commit
lint: test check-codestyle mypy

.PHONY: docs
docs: poetry.lock
	poetry run sphinx-build docs/ docs/_build
