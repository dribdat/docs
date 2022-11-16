# dribdat docs

This repository and its wiki serves to collect helpful advice and curated resources for contributing to open sprints and hackathons, aimed especially at self-starters who are new to organizing or taking part in hackdays and hacknights popular in the open data community. This is a perpetual work-in-progress: new org-hacks and tools pop up all the time, so please contribute your favorites and questions.

🔜 [GOTO Wiki](https://github.com/dribdat/docs/wiki/)

## Contributing

We welcome you to contribute ideas for this guide! Please open an issue, start a Pull Request, or edit the wiki if you have a question or a suggestion that merits discussion. The [community forum](https://forum.opendata.ch) and social media ([@dribdat](https://twitter.com/dribdat)) is also a good place for discussions.

The documentation site for dribdat is built using [Livemark](https://livemark.frictionlessdata.io/). See instructions below for setting this up as a developer:

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
