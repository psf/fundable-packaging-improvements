# Packaging improvements that could be funded

This page lists specific things that

1.  the Python packaging community wants
2.  are fairly well-scoped
3.  would happen much faster if the [Packaging Working
    Group](https://wiki.python.org/psf/PackagingWG) got funding to achieve them
    (through [donations](http://donate.pypi.org/) or grants/directed gifts)

Please contact the Packaging WG by emailing
[`packaging-wg@python.org`](mailto:packaging-wg@python.org) to ask us to estimate how much one of
these improvements would cost; we'll get back to you within a few business
days.

This is roughly prioritized by urgency and impact, but is not a roadmap.

## Foundational tool improvements

### Implement metadata PEPs

The packaging ecosystem relies heavily on access to project metadata. This data
is currently difficult to access, because traditionally it was calculated "on
demand" and only available by downloading the full distribution file. [PEP
643](https://www.python.org/dev/peps/pep-0643/) specifies a mechanism that
allows projects to expose static metadata from source distributions where
available (critically, project name and version are required to be static), and
[PEP 658](https://www.python.org/dev/peps/pep-0658/) allows package indexes to
expose metadata without requiring a download of the full distribution file.

These two PEPs are currently not implemented throughout the packaging ecosystem.
In particular, Warehouse does not yet allow PEP 643 metadata to be uploaded,
and it does not implement PEP 658 for either wheels or source distributions. And
setuptools does not yet implement PEP 643 (while a full implementation involves
complex backward compatibility questions, a simple implementation of static name
and version metadata could be relatively straightforward).

If these two PEPs were implemented as described above, so that they achieve a
critical mass of being available for a significant proportion of package queries,
tools consuming package metadata, such as pip and poetry, could successfully
use that data to simplify and optimise package handling, and adhoc tools would
be able to reliably access package metadata without needing to implement complex
download and build processes.

I am writing this proposal from the point of view of potential consumers of the
relevant metadata. I have not reached out to the setuptools or warehouse projects
to establish if they would be interested in having this work funded, but I believe
that we need to find some way of speeding up adoption of these standards to allow
the ecosystem to move forward.

### Better specifications, toolchain, and services for building distributions

PyTorch, TensorFlow, and many other Python packages (especially
science packages) suffer from cross-platform installability problems,
which affect both users and developers. Packagers and users prefer using
built distributions (usually in the wheel format); publishing built
distributions increases convenience for end users because source code is
pre-compiled, which significantly reduces install time (e.g., from 10+
minutes to several seconds).

Supporting the multifarious Linux platforms is something we've been lagging on;
we are [still finishing up the rollout of
manylinux2010](https://discuss.python.org/t/where-we-are-on-manylinux2010-and-how-that-relates-to-manylinux2014/1987/)
and [recently approved the new standard
manylinux2014](https://www.python.org/dev/peps/pep-0599/). But even so,
packagers will have to build their own wheels to release packages, which can be
fiddly, brittle, and time-consuming.

We'd like help to:

-   [Fully implement & maintain conda-press](https://conda-incubator.github.io/conda-press-docs/)
    Conda-press is a tool that takes conda packages and turns them into wheels, without
    recompiling. This makes it very fast to create a wheel out of an existing package.
    It usually works. However, there have been a variety of bug and maintainence issues
    that require more development (and perhaps a refactor) to address.
-   [Create a generic wheel-building
    service](https://github.com/pypa/packaging-problems/issues/25)  to make
    releases faster and more robust

We need funding for specification research and writing, backend and
frontend development, testing, DevOps/infrastructure/platform services,
user experience work, technical writing for end users, project
management, and community outreach.

#### Port auditwheel to Windows and Mac

The packaging tool [auditwheel](https://github.com/pypa/auditwheel)
"is a command line tool to facilitate the creation of Python wheel
packages for Linux (containing pre-compiled binary extensions) that
are compatible with a wide variety of Linux distributions" and key
standards. It can inspect a wheel, checking whether it is
standards-compliant. It can also repair a wheel. If a wheel depends on
libraries that are not on the system, it can rewrite that wheel and inject
libraries needed, parsing and rewriting ELF data. It can also repair the
relevant `manylinux` tag(s) on a wheel.

However, no such utility exists on Windows, and so package maintainers
on Windows face trouble creating wheels and debugging their
packages. And [the similar utility for Mac
OS](https://pypi.org/project/delocate/) does not share auditwheel's
code and user interface.

Therefore, [developers would like to add Windows and Mac support to
auditwheel.](https://discuss.python.org/t/need-auditwheel-like-utility-for-other-platforms/2028/4)
Porting auditwheel to Windows would make it *much* easier to make
Windows wheels, and porting it to macOS would reduce duplication on
the packaging maintainers' side, and reduce the proliferation of
quirky tools that individual package maintainers need to learn about.

A simpler and more consistent cross-platform workflow will make it
easier for package maintainers to use generic off-the-shelf
automation. More maintainers will be able to leverage available
automation (GitHub Actions, Travis CI, Azure pipelines, and
potentially a future PyPI wheel-building service) to speed up releases
and reduce grunt work. Also, this will especially be useful for
scientific programmers, since they often create Python applications or
libraries that include binaries written in other languages, and wheel
distributions of those packages are prone to complication.

We need funding for backend development, hardware, testing, continuous
integration platform services, technical writing for end users,
project management, and community outreach.

### Robust interoperability testing

We need funding to ensure core packaging tools work well with each other;
currently they aren't seamlessly interoperable. See [the integration-test
project](https://github.com/pypa/integration-test). This will help us get
faster at testing and rolling out bugfixes and features for **all** [Python
packaging and distribution tools](https://packaging.python.org/key_projects):
well-known projects like `pip`, `virtualenv`, and `wheel`, but also all the
downstream projects that depend on them.

### Revamp PyPI API

The [Python Package Index](https://pypi.org/), a key platform for Python developers, has a browser interface, but most people use PyPI by hitting its [API endpoints](https://warehouse.readthedocs.io/api-reference/#available-apis) with client applications such as `pip`. PyPI has [a minimal download API](https://warehouse.readthedocs.io/api-reference/) that does not implement [many features that users have requested](https://github.com/pypa/warehouse/labels/APIs%2Ffeeds). The lack of a full-featured download API in Warehouse (the PyPI codebase) blocks many improvements, including:

-   [Light-bandwidth metadata-only API
    calls](https://github.com/pypa/warehouse/issues/474) and [JSON
    standardization](https://github.com/pypa/pip/issues/7406#issuecomment-583891169)
    that would enable better downloads, installations, dependency resolution
    features, and troubleshooting for `pip` and other clients
-   [RSS feeds](https://github.com/pypa/warehouse/pull/7013) that
    other platforms could reuse to get PyPI updates in user tooling
-   [Security notification feeds](https://github.com/pypa/warehouse/issues/798)
-   [Caching for the bandersnatch mirroring
    client](https://github.com/pypa/warehouse/issues/284)

We'd like to [architect and implement a new Warehouse download API](https://github.com/pypa/warehouse/issues/284) to support these features, and deprecate and decommission the old endpoints. This requires backend development work, technical writing, user experience research, and publicity and coordination work within Python's community.

### Make setuptools the reference implementation of the distutils API

There is a part of the Python standard library
[called](https://docs.python.org/3/library/distutils.html) `distutils`, and some
users directly use it.  [We want users to instead switch to the supported
toolchain](https://github.com/pypa/packaging-problems/issues/127), which uses
`setuptools`, and we want to move all the functionality from `distutils` into
`setuptools`. This requires backend development work, technical writing,
project management, and publicity work within Python's community.

### Provide more standardized editable installations

Developers of Python projects want to be able to use "editable installations"
-- changing the code of applications while simultaneously running those
applications. Right now, the support for that kind of usage is rough and not
standardized across different tools.  [Packaging tools maintainers have rough
plans for how to standardize the feature and support for
it](https://discuss.python.org/t/specification-of-editable-installation/1564)
using `distutils` and `setuptools`. We would like funding for developing a proof of
concept and coordinating subsequent standards changes, tool improvements, and
documentation. This requires backend development work, technical writing, and
coordination and publicity work within Python's community.

### De-duplicate and reorganize setuptools documentation

The [documentation for setuptools](https://setuptools.readthedocs.io/)
has grown messily over time and is difficult to browse and
navigate. Also, the [legacy documentation for
distutils](https://docs.python.org/3/distutils/index.html) and the
current setuptools docs heavily overlap in content. These references
thus trip up even experienced developers who want to understand these
fundamental utilities.

We need funding for several weeks of technical writer work and developer review to:

* de-duplicate the distutils and setuptools documentation, making the latter independent of the former
* re-organize the setuptools documentation

### Add support for reproducible builds to setuptools

[Reproducible builds](https://en.wikipedia.org/wiki/Reproducible_builds) allow
developers to independently verify that a distributed software package was not
tampered with. Since a considerable number of the Python packages use
`setuptools`, adding support for reproducible builds to this build backend
can help to improve security in Python ecosystem as a whole.
Some preliminary works that can help structuring this activity are available (see
[pypa/setuptools#2133](https://github.com/pypa/setuptools/issues/2133),
[pypa/setuptools#1512](https://github.com/pypa/setuptools/pull/1512) and
[pypa/wheel#362](https://github.com/pypa/wheel/issues/362)), however the
effort was never concluded. Funding would be used finalize the development of
this feature.

Import tasks are:
- Support `SOURCE_DATA_EPOCH` environment variable for both sdists and wheels.
- Make both sdist and wheels independent of umask.
- Ensure that C/C++-extensions compiled with setuptools are reproducible.
- Document the process of verifying a sdist or wheel.

This project might require coordination with other tools in the ecosystem
(e.g. `wheel`).

### Audit and update package metadata

If we [audit and update PyPI metadata for existing projects based on
already-uploaded
artifacts](https://github.com/pypa/warehouse/issues/474#issuecomment-370986838),
we can publish information about what packages depend on each other and on
certain environments, and ensure a high-quality API for many tools to reuse and
build upon. The current PyPI upload API relies on the upload client extracting
the metadata and supplying it with the first upload request, and that isn't a
valid assumption for older upload clients. Currently, our constraint is a
combination of developer time, compute resources, and privileged backend
database access; funding would break this bottleneck.

### Improve user experience of packaging

User experience research, and UX and development implementation work, would [make
it easier for packagers to create configuration
files](https://github.com/pypa/packaging-problems/issues/1). We aim to use the
UX research work from [improvements in pip's user
experience](https://wiki.python.org/psf/Fundable%20Packaging%20Improvements#Improve_pip_user_experience)
and build on them to improve the larger experience of packaging for Python in
general.

### Improve specificity of license classifiers

Our packaging ecosystem relies on [a particular structured data format
(classifiers)](https://pypi.org/classifiers/) to indicate a package's legal
license. However, our current system [allows for ambiguity that makes some
downstream data display incoherent or very difficult, and doesn't allow for some
license specificity that downstream consumers
need](https://github.com/pypa/warehouse/issues/2996)
([Libraries.io](https://libraries.io/) and similar projects). Fixing this is a
fairly small project, involving Python development, public communications,
project management, and potentially a few hours of legal counsel for review.

### Standardize and implement a lockfile format

`pip` currently uses `requirements.txt` to specify dependencies; it can specify
__versions__ of packages but not __hashes__. The [newer pipfile
format](https://github.com/pypa/pipfile) can include hashes, which some users
prefer. But `pip` [doesn't yet
support](https://github.com/pypa/pip/issues/4732) `pipfile`, so many users are
blocked from using hashes to better secure their Python runtimes. We have [made
some progress toward standardizing an interoperable lockfile
format](https://github.com/uranusjr/lock-file), but we need to finish [that
design standardization and consensus-gathering
work](https://discuss.python.org/t/structured-exchangeable-lock-file-format-requirements-txt-2-0/876/)
and implement it in `pip`, `pipenv`, and related tools. Other attempts reached the PEP
stage [^pep-650][^pep-665], but ultimately were rejected. We'd need Python
engineering work and project management to develop and deploy this.

[^pep-650]: https://peps.python.org/pep-0650/
[^pep-665]: https://peps.python.org/pep-0665/

### Package preview feature for PyPI

Right now, there are ways for package maintainers to test and share draft
versions of their upcoming releases, but they cause friction and confusion. So
we want to add [staged releases -- a temporary state that a release can be in,
where PyPI __has__ it and can evaluate it, but hasn't __published__ it
yet](https://github.com/pypa/warehouse/issues/726).

This will:

-   let project owners/maintainers
    preview/[test](https://github.com/pypa/warehouse/issues/720)
    how their package metadata displays on the website, and review where
    their fresh releases are out of compliance with site and
    interoperability requirements (preventing the problem of
    [maintainers wanting to re-upload removed files](https://github.com/pypa/packaging-problems/issues/74))
-   help cross-platform package maintainers
    [coordinate dozens of wheels built on multiple machines](https://github.com/pypa/warehouse/issues/4056) for simultaneous release
-   [Provide an interoperability check for toolchain developers, and a testing site for people learning packaging](https://github.com/pypa/warehouse/issues/2286)
-   [Simplify packagers' upload configuration files](https://github.com/takluyver/flit/issues/125)
-   reduce complexity that currently forces maintainers to use
    [confusing "dev" or prerelease version numbers](https://github.com/pypa/warehouse/issues/5707)
-   [Improve security of package uploads, by allowing maintainers to scope upload API tokens to the newly staged package](https://github.com/pypa/warehouse/issues/6378)
-   [Prevent package name conflicts](https://github.com/pypa/packaging-problems/issues/114)
-   [Streamline infrastructure maintenance and confusing documentation by letting us take down the separate test.pypi.org staging site](https://github.com/pypa/warehouse/issues/918)
-   Provide pre-release warnings to maintainers of packages that [fail metadata
    checks](https://github.com/pypa/packaging-problems/issues/264) (such as
    rejecting or warning for [packages without Python requirements
    metadata](https://github.com/pypa/warehouse/issues/3889), or [manylinux
    wheels that fail auditwheel
    checks](https://github.com/pypa/warehouse/issues/5420) -- as we increase
    the packaging ecology's strictness regarding metadata standards compliance,
    during the intermediate period where we're warning maintainers/owners about
    failing strictness checks but not yet blocking releases on those new
    stricter checks, the package preview feature will help us provide soft
    warnings.

We'll need database support for understanding the release state ("is
this published or not"), user experience and developer support, and
testing, security, infrastructure, and project management support.

### Feature flag system on PyPI

It's difficult to roll out new features gradually to PyPI's test site or
to selected test users. A
[feature flag system](https://github.com/pypa/warehouse/issues/5869) would help us do targeted outreach to particular groups of
users, deploy more confidently, and roll back changes when needed. We'd
need user experience, front and backend engineer, data analytics, and
project management support to develop and deploy this.

### User support ticket system

Python packagers who need help currently create
[Sourceforge](https://sourceforge.net/p/pypi/support-requests/)
and [GitHub](https://github.com/pypa/pypi-support/issues) tickets,
email mailing lists, tweet at maintainers, and so on. A
[unified user support ticket system](https://github.com/pypa/warehouse/issues/3231#issuecomment-405561741), integrated into Warehouse, would:

-   help managers, entrepreneurs, and academics
    [reserve specific package names](https://github.com/pypa/warehouse/issues/2082)
-   [support username changes](https://github.com/pypa/warehouse/issues/1190)
-   give users [a reporting system to quickly flag malware and spam](https://github.com/pypa/warehouse/issues/2982)
-   provide a [transfer system for abandoned/unmaintained projects](https://github.com/pypa/warehouse/issues/1506)
-   reduce work for PyPI's core developers who currently have to sift
    through user support issues to find bug reports and feature requests
-   enable PyPI admins to better delegate support and moderation work to
    volunteers

We need funding for backend and frontend development, testing and
security checks, DevOps/infrastructure/platform services (including
API/email integration), user experience work, technical writing for end
users, project management, and community outreach.

### Architecture to support alternative authentication methods in packaging tools

Python packaging tools that interact with package indexes, such as pip ([pypa/pip#4475](https://github.com/pypa/pip/issues/4475)) and twine ([pypa/twine#362](https://github.com/pypa/twine/issues/362)), currently only have simple authentication support to secure private sources, such as [basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). Open source tool maintainers acknowledge that, when using third-party indices, sometimes organisational policies require stronger authentication methods, such as single sign-on. We believe it’s beneficial to develop a pluggable Python library that can be depended by the packaging tools to provide additional authentication methods. But we lack both the use case and domain knowledge in the area. We are looking for funding and expertise support from organisations.

We are interested in developing a shared interface and implementation for various alternative authentication methods. Support can be developed for both tools (and maybe more), so organisations can choose to install them to be able to use e.g. Kerberos to secure their private package indexes. The work involved would include development, research, project management, and technical writing work towards the following tasks:

* Survey various authentication methods, and how they can be implemented as a pluggable library.
* Develop an interface that tools (e.g. pip) can implement to detect authentication method support, and call into the library that provides it.
* Develop and maintain libraries that implement the various auth methods for users to install when support is needed.

### Release wheel 1.0 and provide a public API

The wheel project, the official binary distribution format for Python,
is [approaching version
1.0](https://mail.python.org/archives/list/distutils-sig@python.org/thread/X2NNYA25EBAQYTPKRSJASPIBY6YT5VKB/). The
[milestone](https://github.com/pypa/wheel/milestone/1) includes
[Provide a public API: "Since there is clearly a need for wheel to
function as a library as well, a public API should be defined and
documented."](https://github.com/pypa/wheel/issues/262) The stability
implied by the 1.x version number, and the public API, will improve
other tools' ability to call and reuse `wheel`, thus reducing
duplication and improving other toolmakers' ability to move faster and
maintain their codebases more easily. We would like support for design
and implementation, community coordination, and technical writing.

## Security improvements and prerequisites

### Improved support for Multi-Factor Authentication in Warehouse (PyPI)

To mitigate account takeover attacks, where attackers upload malicious code in existing popular packages, we need to continue improving our support for MFA and use of API tokens instead of password-based auth.  This work would involve research, development, and technical writing to finalise and implement existing proposals including:

- [Require API Tokens for upload if MFA is enabled](https://github.com/pypa/warehouse/issues/7265) (upload currently bypasses MFA)
- [Improve 2FA Account Recovery request fulfillment process](https://github.com/pypa/pypi-support/issues/796)
- [other issues tagged `tokens`](https://github.com/pypa/warehouse/labels/tokens)

Once these technical prerequisites are satisfied, we would be able to [revisit discussion of MFA *policy*](https://discuss.python.org/t/require-mfa-on-pypi/12077) - including encouraging MFA requirements for popular packages, or even mandating it for all users as some other package registries are considering.  This subsequent work would involve additional development, project management, and community engagement as we determine and implement authentication policy, including ongoing user support to handle a much greater frequency of account recovery requests.

### System to label projects on PyPI with administrative statuses/attributes

To scale up our anti-abuse moderation and help package maintainers with
security response, we need to be able to, for instance, mark a release
as deprecated or a project as unsupported. This means we need a generic
system to add, edit, and remove administrative attributes ("flags" or
"statuses") to individual projects and releases. We need support to do
the architectural design to implement this. (See
[notes from this meeting](https://wiki.python.org/psf/PackagingWG/2019-03-22-Warehouse).)

### Security notifications for vulnerable packages

To keep PyPI's users secure, we want to give them
[an opt-in communication channel to hear about security vulnerabilities for the packages they use](https://github.com/pypa/warehouse/issues/798). Implementing this would also give us
architectural support to __warn or prevent__ `pip` users who try to
install a PyPI package that's been found to be broken or malware. We
need funding for user experience work, development, testing,
infrastructure, potentially platform services (e.g., SMS), and community
outreach.

### Check for insecure maintainer email addresses

[Recent research on weaknesses in the `npm` supply chain](https://arxiv.org/abs/2112.10165)
identified 2,818 maintainer email addresses at expired domains, affecting 8,494 packages.
Such weaknesses can be mitigated by multi-factor authentication, but generally make
targeted account hijacking trivially easy.

Because registration expiry dates are public via whois records, PyPI could warn maintainers
with email addresses at soon-to-expire domains.  As a further enhancement, PyPI could block
password resets via any email address at a domain which has expired and been re-registered
*after* the 30 day renewal grace period.  This gives defenders a natural time-based advantage
over attackers.

Funding would be used for backend development, security engineering, project management,
system administration, outreach to package maintainers, and ongoing operational costs.


### Productionize Malware Detection
Since anyone can upload a package to PyPI, malicious users might upload malware, which would then harm users. To mitigate this risk, PSF [previously obtained funding](https://github.com/python/request-for/blob/master/2019-Q4-PyPI/RFP.md) to add some malware detection in [Warehouse](https://github.com/pypa/warehouse) in late 2019, but the goals for [the relevant milestone](https://github.com/python/request-for/blob/master/2019-Q4-PyPI/RFP.md#milestone-2---systems-for-automated-detection-of-malicious-uploads) were more ambitious than funding allowed for. The malware detection system is currently in limbo: an interesting prototype with limited practical impact because of the astounding number of false-positives. To protect users from malware, we still need to:
- [Make Malware Verdicts Auditable](https://github.com/pypa/warehouse/issues/7421) - Right now, verdicts are removed once the associated project/package is removed. We need to change the backend to retain past verdicts so we can evaluate and improve the efficacy of this system.
- [Detect packages being published with typo'ish names](https://github.com/pypa/warehouse/issues/4998) - Add a typosquatting check.
- [Implement a more robust malware detector](https://github.com/pypa/warehouse/issues/7748) - One current check relies on simple pattern matching with YARA. A better approach requires parsing the package code into an AST.
- [YARA rules for setup.py not ignoring comments](https://github.com/pypa/warehouse/issues/7475) - Related to the above issue.
- Release a canonical dataset for developers to write their checks against, and iterate on to improve detection accuracy.

We also want to set up a partnership with [VirusTotal](https://www.virustotal.com/) or a similar third-party virus checking service during the check development to scan every uploaded package. Integration with a third-party virus scanner is low-hanging fruit that could move the needle on PyPI package security.

Funding would be used for backend development, security engineering, project management, system administration, and publicity to stakeholders. Ideally, AV integrations would be donated by the vendors.

# Items that have now been funded

Some TODOs that were on this page have now received funding!

## Foundational tool improvements

### Finish dependency resolver for pip

__([This is now funded](https://pyfound.blogspot.com/2019/11/seeking-developers-for-paid-contract.html) and
[we hired people to work on this project](https://github.com/python/request-for/blob/master/2020-pip/RFP.md). The [new resolver is in beta](https://blog.python.org/2020/07/upgrade-pip-20-2-changes-20-3.html).)__

We're partway through a next-generation rewrite of the dependency
resolver within pip, Python's package download and installation tool.
The project ran into massive technical debt, but the refactoring is
nearly finished and prototype functionality is in alpha now.
([In-depth explanation by Sebastian Awwad of the problem & our approach](https://docs.google.com/document/d/1x_VrNtXCup75qA3glDd2fQOB2TakldwjKZ6pXaAjAfg/edit),
[lead developer Pradyun Gedam's initial plan](https://gist.github.com/pradyunsg/5cf4a35b81f08b6432f280aba6f511eb),
[2017 status updates](https://pradyunsg.me/gsoc-2017/), and [GitHub issue #988 tracking progress](https://github.com/pypa/pip/issues/988) and [June 2019 status update](https://pradyunsg.me/blog/2019/06/23/pip-update/), and
[issue #6536 for planning rollout](https://github.com/pypa/pip/issues/6536).)

Funding would support user experience, communications/publicity, and
testing work (including developing robust testing/CI infrastructure) as
well as core feature development and review.

We need to finish the resolver because so many other improvements are
blocked on it:

* [adding an "upgrade-all" command to pip](https://github.com/pypa/pip/issues/4551)
* [warning when trying to download or build wheels from incompatible set of packages/requirements](https://github.com/pypa/pip/issues/5497)
* [adding a no-implicit-upgrades strategy](https://github.com/pypa/pip/issues/4745)
* [making PyPI and pip enforce metadata compliance more strictly](https://github.com/pypa/packaging-problems/issues/264)
* [warning the user when uninstalling a package that other packages depend on](https://github.com/pypa/pip/issues/4681)
* [properly respecting constraints](https://github.com/pypa/pip/issues/6495)
* [recording requested and installed extras](https://github.com/pypa/packaging-problems/issues/215)
* [option to show what versions of packages are currently available](https://github.com/pypa/pip/issues/53)
* [listing packages' dependencies and dependents on PyPI](https://github.com/pypa/packaging-problems/issues/54)
* [minimizing duplication of work between pip and pipenv](https://mail.python.org/archives/list/distutils-sig@python.org/thread/2QECNWSHNEW7UBB24M2K5BISYJY7GMZF/#2QECNWSHNEW7UBB24M2K5BISYJY7GMZF)
* [better pipenv functionality](https://github.com/pypa/pipenv/issues?q=is%3Aopen+is%3Aissue+label%3A%22Category%3A+Dependency+Resolution%22)
* [package namespace support](https://discuss.python.org/t/namespace-support-in-pypi/1609/35)
* [moving more code out of Python's standard library so we can release improvements faster](https://discuss.python.org/t/if-python-started-moving-more-code-out-of-the-stdlib-and-into-pypi-packages-what-technical-mechanisms-could-packaging-use-to-ease-that-transition/1738/24)

and it would fix so many dependency issues for our users:

* [Django installation conflict](https://github.com/pypa/pip/issues/4907)
* [cherrypy/six/cheroot installation conflict](https://github.com/pradyunsg/zazo/issues/2)
* [Spyder downgrade requirement](https://github.com/pypa/pip/issues/5043)
* [boto3/bravado dependency failure](https://github.com/pradyunsg/zazo/issues/4)
* [Ansible/PyOpenSSL/cryptography failure](https://github.com/pypa/pip/issues/5313)
* [extras installation failure](https://github.com/pypa/pip/issues/4957)
* [extras upgrade failure](https://github.com/pypa/pip/issues/4391)
* [breaking installed packages](https://github.com/pypa/pip/issues/6494)
* [elasticsearch/requests failure](https://github.com/pradyunsg/zazo/issues/14)
* [hatch, another packaging tool](https://github.com/ofek/hatch/issues/47)

And in our larger ecology, this causes installation problems for:

* [conda's compatibility with pip](https://github.com/conda/conda/issues/8657)
* [the Servo browser engine](https://github.com/servo/servo/issues/10611)
* [numpy and scipy](https://github.com/pypa/pip/issues/4582)
* [Canonical's DevOps tool Juju](https://github.com/juju/python-libjuju/issues/45)
* [a Cap'n Proto implementation](https://github.com/antocuni/capnpy/issues/16)
* [toil, awscli, and boto3](https://github.com/DataBiosphere/toil/issues/2230)
* [the Mozilla website & icalendar](https://github.com/mozilla/bedrock/issues/5967)
* [certbot, in the past and possibly the future](https://github.com/certbot/certbot/issues/5195)
* [TurboGears](https://github.com/TurboGears/tg2devtools/issues/13)
* [a JIRA API client library](https://github.com/pycontribs/jira/pull/744)
* [a WebSocket protocol test suite](https://github.com/crossbario/autobahn-testsuite/issues/55)
* [Robot Operating System tooling](https://github.com/gerkey/ros1_external_use/issues/7)

### Improve pip user experience

__This is now funded, thanks to
[the Chan Zuckerberg Initiative](https://chanzuckerberg.com/eoss/proposals/improving-user-experience-and-debuggability-of-pip-for-all-python-users/) and
[Mozilla Open Source Support](https://www.mozilla.org/en-US/moss/).__

`pip`'s user experience needs to improve by providing
[better error messages](https://github.com/pypa/pip/milestone/25)
and prompts, logs, output, and reporting, and becoming more consistent
across features, to fit the user's mental model better, make hairy
problems easier to untangle, and reduce unintended data loss. `pip`'s
maintainers have [a list of TODOs](https://github.com/pypa/pip/milestone/10) and need funding so that user experience researchers, UX
designers, developers, and technical writers can spend dedicated time
addressing them.
