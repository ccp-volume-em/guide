# Making software public

Publishing research software is more than making a repository visible. A useful
public release should be installable, documented, licensed, citable and free of
information that should remain private. The recommendations below assume that
the software has already been tested, reviewed and versioned as described in
[Software Development](software.md).

## Before publishing

Review the complete repository history as well as the current files. Removing a
secret from the latest commit does not remove it from earlier commits.

Before the first public release:

* Confirm that all contributors and, where relevant, their employers or funders
  agree that the software can be released.
* Remove credentials, access tokens, private URLs, personal data, patient data,
  confidential issue discussions and data that you do not have permission to
  redistribute. Revoke and replace any credential that has ever been committed.
* Check the licences of source code, models, example data, fonts, icons and
  other third-party material. Record their copyright and attribution notices.
* Add a `README`, a licence, installation and usage instructions, contributor
  guidance, a citation file and a way to report problems.
* Test installation in a clean environment and run the automated test suite.
  Test the built package, not only the source checkout.
* Choose a release version and prepare release notes describing important
  changes, known limitations and any compatibility breaks.

Avoid committing large datasets, model weights and generated build files to the
source repository. Deposit large research outputs in an appropriate public repository
and link to them using a persistent identifier.

## GitHub

A public [GitHub repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories)
makes the source and its history discoverable and gives users a place to report
issues and contribute changes. Before changing a private repository to public,
inspect branches, tags, pull requests, Actions logs and release assets: some of
these also become visible. GitHub documents the
[consequences of changing repository visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility).

Suggested contents for the repository:

* `README.md`: purpose, status, supported platforms, installation, a minimal
  example, links to full documentation, support and citation instructions.
* `LICENSE` or `LICENSE.md`: the full licence text.
* `CITATION.cff`: title, authors, ORCID identifiers, version, release date and
  DOI, when available. GitHub uses this file to display a **Cite this
  repository** link; see its
  [citation-file guidance](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-citation-files).
* `CONTRIBUTING.md`: how to set up a development environment, run tests and
  submit a change.
* `CODE_OF_CONDUCT.md`: expected behaviour and a route for confidential
  reporting.
* `SECURITY.md`: supported versions and how to report a vulnerability without
  first disclosing it publicly.
* A dependency or environment specification, with direct dependencies and
  supported language versions declared accurately.

Use an organisation-owned repository for collaborative projects when possible,
so that stewardship does not depend on one person's account. Give at least two
maintainers the permissions needed to make a release and enable multi-factor
authentication and branch protection.

### Create a release

Create a release from a reviewed commit on the default branch:

1. Update the version, changelog, citation metadata and documentation.
2. Run tests and build the documentation and distribution artefacts in a clean
   environment.
3. Create an annotated Git tag using the project's version convention, for
   example `v1.2.0`.
4. Create a GitHub Release from that tag and add release notes.
5. Publish packages and container images from the same tag, then test that
   users can install them using the documented commands.

[GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)
associate a fixed Git tag with release notes and downloadable assets. Do not
move or reuse a published tag. If a release is faulty, publish a new patch
release and explain the problem.

For long-term preservation and citation, connect the repository to a research
archive such as [Zenodo](https://help.zenodo.org/docs/github/). Archive each
release and add the resulting version-specific DOI to `CITATION.cff`. A concept
DOI can refer to the software as a whole, while a version DOI identifies the
exact software used in an analysis; prefer the version DOI when reproducibility
requires a particular release.

## PyPI package

Publishing a Python library or application on the
[Python Package Index (PyPI)](https://pypi.org/) allows users to install it with
`pip`. Follow the maintained
[Python Packaging User Guide](https://packaging.python.org/en/latest/tutorials/packaging-projects/)
rather than invoking `setup.py` directly.

Define the build system and package metadata in a single definition using `pyproject.toml`. Metadata should
include:

* a unique distribution name and version;
* a short description and `README` as the long description;
* the licence expression and licence files;
* required Python version and runtime dependencies;
* authors or maintainers;
* project URLs for source, documentation, issues and changelog;
* classifiers and optional dependencies where useful.

Build both a source distribution and a wheel, then check their metadata:

```console
python -m pip install --upgrade build twine
python -m build
python -m twine check --strict dist/*
```

Inspect the archive contents and install the wheel into a new virtual
environment. If the project has compiled extensions, provide wheels for the
supported combinations of operating system, architecture and Python version,
and test each build.

Use [TestPyPI](https://packaging.python.org/en/latest/guides/using-testpypi/)
to exercise a new packaging workflow before the production release. PyPI does
not permit a published file or version to be replaced, so corrections require a
new version.

For automated releases, prefer
[PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/). It uses
short-lived identity tokens from the CI service instead of storing a long-lived
PyPI token in repository secrets. Restrict the publishing job to protected tags
or a protected release environment, and require review where appropriate.
Build once, test those exact artefacts, and promote the same files to the
production index.

After publishing, verify the package page and test the normal user workflow:

```console
python -m pip install your-package
python -c "import your_package"
```

## Licence

Copyright exists even when no licence file is present. A public repository
without a licence can be viewed and forked under the hosting platform's terms,
but that does not generally give others permission to use, modify or distribute
the software. See [Choose a License](https://choosealicense.com)
for an accessible explanation.

Choose the licence before accepting external contributions, and obtain agreement
from all copyright holders. Consider:

* compatibility with the licences of dependencies and incorporated code;
* whether derived works may be distributed under other terms (a permissive
  licence) or must remain open under compatible terms (a copyleft licence);
* requirements imposed by an employer, funder, consortium or target ecosystem;
* patent provisions, attribution requirements and the intended commercial use.

Common choices include BSD-3-Clause, MIT and Apache-2.0 as permissive licences,
and GPL-3.0 as a strong copyleft licence. This list is not legal advice; consult
your institution's research software or legal team when ownership or licence
compatibility is unclear. Use a standard
[OSI-approved licence](https://opensource.org/licenses) without changing its
text, and include its exact SPDX identifier in package metadata.

A repository may need separate licences for software, documentation, data and
model weights. State clearly which files each licence covers. Creative Commons
licences may suit documentation and data but are
[not recommended for software](https://choosealicense.com/non-software/).
Retain third-party notices and make sure example data can legally be published.

## Statements about AI use

Generative-AI tools may influence source code, tests, documentation, images or
translations. The human contributors and maintainers remain responsible for
everything released: AI systems should not be listed as authors or copyright
holders.

Follow the policies of your institution, funder, publisher and software
community. When AI use was substantive, add a short statement to the repository,
release notes or associated paper. Record enough information to make the role of
the tool understandable, without publishing confidential prompts or sensitive
data. A statement can identify:

* the tool and model, including the version or access date where known;
* which parts of the work it assisted with;
* whether generated output was modified;
* how a human reviewed, tested and validated the result.

For example:

> Generative AI was used to suggest docstrings and unit-test cases. All
> suggestions were reviewed and edited by the maintainers, and the resulting
> software passed the project's automated tests. No confidential source code or
> research data was provided to the service.

If no generative-AI tool contributed to the release, a statement is normally
unnecessary unless a relevant policy explicitly requires one.

Do not submit personal, confidential, security-sensitive or unpublished
material to an external AI service unless its data handling terms and your
project approvals permit this. Treat generated code like any other external
contribution: understand it, review it for security and bias, test it, and check
for licence or attribution concerns before release. AI output can be plausible
but incorrect, and an AI-use statement does not replace quality assurance.
