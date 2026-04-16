# Software Development

For collaborative software development, we recommend the following guidance.

## Collaborative development

### Platform
Use a cloud platform (e.g. [GitHub](https://github.com)), using repositories for code and documentation, project management and communication.
We also recommend using GitHub Issues so users can report issues and bugs, feature-level branching and Pull Requests.

### Version Control
Use version control to track changes, as well as versioning of releases. As a rule a new release is typically made when new code is made available to users.
We recommend using git, and tagging releases with unique versioning in the format Major.Minor.Patch (a detailed description can be found [here](https://en.wikipedia.org/wiki/Software_versioning)) and prefixing versions with the letter v (e.g. v1.0.1).
More information on symatic versioning: [Semver](https://semver.org/) and effort-based versioning: [EffVer](https://jacobtomlinson.dev/effver/)

### Code Reviews
Perform code reviews by a colleague at important stages of development.

## Documentation
Both user documentation and developer documentation are important, so users can use the software and developers can understand and contribute to the code.
We recommend using [markdown files (.md)](https://en.wikipedia.org/wiki/Markdown) and automated documentation deployment (e.g. [mkdocs](https://www.mkdocs.org/)) to GitHub pages.
For diagrams we like using [mermaid](https://en.wikipedia.org/wiki/Mermaid_(software)).

### Docstrings
Code documentation is provided using docstrings.
There are different styles of this including the [Numpy style](https://numpydoc.readthedocs.io/en/latest/format.html).
Different tools are available to help generate these in the code, and subsequently to automatically generate documentation from these.

### Comments
Comments are an important even more granular part of development documentation.
Avoid repeating input and output arguments without information beyond the names and types.
Here it's more important to describe the function and to explain the why questions.
Inline comments are useful anywhere where it's not intuitively obvious from code what or why something is done.

## Containerization
For reproducibility and ease of installation, we recommend using creating containers (e.g. [Docker](https://www.docker.com/)) to create a consistent environment for users.

## Testing
For python testing we recommend using [pytest](https://docs.pytest.org/) together with [tox](https://tox.wiki/) and connecting this with an automated GitHub Action triggered by a pull request, push or release.
Additionally, code coverage can give a useful indicator of how much of the code is covered by tests, automated tools like [codecov](https://codecov.io/) can be used for this.

## Maintenance
Maintenance is an important part of software development, to keep it functional and relevant and so others can keep using it.
We recommend regular maintenance, to keep it up to date especially considering dependencies and security.
But this also includes bug fixes and feature updates, and updating documentation as needed.
We also recommend to make this as efficient as possible considering limited resources, so ensuring good continued maintainability.
There are tools for automated dependency updates, such as [Dependabot](https://github.com/dependabot) and [Renovate](https://docs.renovatebot.com/), including taking care of security vulnerabilities.

## Support
Besides Github Issues, for further user support we recomment connecting to a single suitable community forum (e.g. [image.sc](https://forum.image.sc/) for image analysis related software) and providing a link to this forum in the documentation.
This also allows the community to support each other and share knowledge.

## Further resources
- A collection of resources for python tooling: [ARC python tooling](https://github-pages.arc.ucl.ac.uk/python-tooling/)
- A comprehensive guide to reproducible data science, including software development and documentation: [Turing way book](https://book.the-turing-way.org)
- A tutorial on using git for version control: [Git software carpentry](https://swcarpentry.github.io/git-novice/)
- A tutorial on intermediate python development, including testing and documentation: [Carpentries Incubator](https://carpentries-incubator.github.io/python-intermediate-development/)
- Documentation on using Docker for containerization: [Docker documentation](https://docs.docker.com/)

- CCP-volumeEM: Good practices for image analysis software with Dr Kimberly Meechan [YouTube video](https://www.youtube.com/watch?v=HHJJZ79KC4Y)

