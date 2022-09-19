# dribdat docs

Documentation site for dribdat, built using Livemark.

See https://livemark.frictionlessdata.io/

## Installation

Install [Python Poetry](https://python-poetry.org/) and then install the dependencies:

`poetry install`

Enter the Poetry environment:

`poetry shell`

If all is good you can start the local server:

`livemark start`

Open http://localhost:7000/docs/ in your browser.

## Deployment

This command updates the gh-pages branch on GitHub:

`livemark build && ghp-import -p -f -o docs/`