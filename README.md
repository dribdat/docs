# Dribdat docs

This repository and [wiki drafts](https://github.com/dribdat/docs/wiki) collect helpful advice on using Dribdat. See the web version at https://dribdat.cc and for more curated resources, visit [awesome-hackathons](https://github.com/dribdat/awesome-hackathon?tab=readme-ov-file#awesome-hackathon).
This is a perpetual work-in-progress: new org-hacks and tools pop up all the time, so please contribute your favorites and questions.

📖 100 [GOTO HANDBOOK](https://dribdat.cc)
🕹️ 200 [GOTO FORUMS](https://github.com/orgs/dribdat/discussions)
🔜 300 [GOTO WIKI](https://github.com/dribdat/docs/wiki/)

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

We are currently deploying this inside of a tidy [GitHub Action](.github/workflows/livemark-gh-pages.yml), no need for manual upload. This command force updates the gh-pages branch on GitHub:

`livemark build && ghp-import -p -f -o docs/`

(Use only if there's an issue with the Action!)
