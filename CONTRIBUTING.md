# Contributing to Ohala

Start with an issue labelled `good first issue` or `help wanted`. Comment to claim it. Claims lapse after 14 days without a draft PR.

## Your own work

Code, tests and translations must be your own work.

This project is a learning ground for LND. Reviewers give their time to people who are learning, and to code whose author can explain every line.

Using a chatbot to learn outside the repo is your business. What you submit must be your own writing.

## Commits

Every commit message is one line: a short subject, no body, no trailers. Follow LND's `pkg: short summary` style. CI fails any longer message.

## Dev setup

The code skeleton is not built yet. When it lands, you will need Go and Polar (or `make dev` for regtest). The starter issues build the skeleton in order.

## Style

* `gofmt` and `go vet` must be clean.
* One logical change per PR.
* Tests for behaviour changes.
* New user-facing strings go in `locales/en.json`.

## Pull requests

Fill in the template and tick every box. A maintainer reviews every PR before merge.

## Conduct

Be kind. The [Code of Conduct](CODE_OF_CONDUCT.md) applies everywhere in this project.
