# Software development

For collaborative software development, we recommend using the following:

## Collaborative development

### Platform
Use a cloud platform (e.g. [GitHub](https://github.com)), using repositories for code and documentation, project management and communication.
We also recommend using GitHub Issues so users can report issues and bugs, feature-level branching and Pull Requests.

### Version Control
Use version control to track changes, as well as versioning of releases. We recommend using git, and tagging releases with semantic versioning in the format Major.Minor.Patch (a detailed description can be found [here](https://en.wikipedia.org/wiki/Software_versioning)).

### Code Reviews
Perform code reviews by a colleague at important stages of development.

## Documentation
We recommend using [markdown files (.md)](https://en.wikipedia.org/wiki/Markdown) and automated documentation deployment (e.g. [mkdocs](https://www.mkdocs.org/)) to GitHub Pages. For diagrams we like using [mermaid](https://en.wikipedia.org/wiki/Mermaid_(software)).

### Comments
Comments are an important part of development documentation. Avoid repeating input and output arguments without information beyond the names and types. It's more important to describe the function and to explain why the implementation may have been chosen.
Inline comments are useful anywhere where it's not intuitively obvious from code what or why something is done.

## Testing
For python testing we recommend using [pytest](https://docs.pytest.org/) together with [tox](https://tox.wiki/), and connecting this to a GitHub Action triggered by a push or release.

## Containerization
For reproducibility and ease of installation, we recommend using creating containers (e.g. [Docker](https://www.docker.com/)) to create a consistent environment for users.

## Support
Besides GitHub issues, for further user support we recomment connecting to a single suitable community forum (e.g. [image.sc](https://forum.image.sc/) for image analysis related software, and providing a link to this forum in the documentation.
This also allows the community to support eachother and share knowledge.
