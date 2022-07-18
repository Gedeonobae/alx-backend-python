##  Unittests and Integration Tests

﻿![](Top.png)

Unit testing is the process of testing that a particular function returns expected results for different set of inputs. A unit test is supposed to test standard inputs and corner cases. A unit test should only test the logic defined inside the tested function. Most calls to additional functions should be mocked, especially if they make network or database calls.

The goal of a unit test is to answer the question: if everything defined outside this function works as expected, does this function work as expected?

Integration tests aim to test a code path end-to-end. In general, only low level functions that make external calls such as HTTP requests, file I/O, database I/O, etc. are mocked.

Integration tests will test interactions between every part of your code.

Execute your tests with

```sh
$ python -m unittest path/to/test_file.py
```

# Requirements

## General

- All your files will be interpreted/compiled on Ubuntu 18.04 LTS using python3 (version 3.7)
- All your files should end with a new line
- The first line of all your files should be exactly #!/usr/bin/env python3
- A README.md file, at the root of the folder of the project, is mandatory
- Your code should use the pycodestyle style (version 2.5)
- All your files must be executable
- All your modules should have a documentation (python3 -c 'print(__import__("my_module").__doc__)')
- All your classes should have a documentation (python3 -c 'print(__import__("my_module").MyClass.__doc__)')
- All your functions (inside and outside a class) should have a documentation (python3 -c 'print(__import__("my_module").my_function.__doc__)' and python3 -c 'print(__import__("my_module").MyClass.my_function.__doc__)')
- A documentation is not a simple word, it’s a real sentence explaining what’s the purpose of the module, class or method (the length of it will be verified)
- All your functions and coroutines must be type-annotated.

## Required Files

- utils.py

```sh
#!/usr/bin/env python3
"""Generic utilities for github org client.
"""
import requests
from functools import wraps
from typing import (
    Mapping,
    Sequence,
    Any,
    Dict,
    Callable,
)

__all__ = [
    "access_nested_map",
    "get_json",
    "memoize",
]


def access_nested_map(nested_map: Mapping, path: Sequence) -> Any:
    """Access nested map with key path.
    Parameters
    ----------
    nested_map: Mapping
        A nested map
    path: Sequence
        a sequence of key representing a path to the value
    Example
    -------
    >>> nested_map = {"a": {"b": {"c": 1}}}
    >>> access_nested_map(nested_map, ["a", "b", "c"])
    1
    """
    for key in path:
        if not isinstance(nested_map, Mapping):
            raise KeyError(key)
        nested_map = nested_map[key]

    return nested_map


def get_json(url: str) -> Dict:
    """Get JSON from remote URL.
    """
    response = requests.get(url)
    return response.json()


def memoize(fn: Callable) -> Callable:
    """Decorator to memoize a method.
    Example
    -------
    class MyClass:
        @memoize
        def a_method(self):
            print("a_method called")
            return 42
    >>> my_object = MyClass()
    >>> my_object.a_method
    a_method called
    42
    >>> my_object.a_method
    42
    """
    attr_name = "_{}".format(fn.__name__)

    @wraps(fn)
    def memoized(self):
        """"memoized wraps"""
        if not hasattr(self, attr_name):
            setattr(self, attr_name, fn(self))
        return getattr(self, attr_name)

    return property(memoized)
```


- client.py

```sh
#!/usr/bin/env python3
"""A github org client
"""
from typing import (
    List,
    Dict,
)

from utils import (
    get_json,
    access_nested_map,
    memoize,
)


class GithubOrgClient:
    """A Githib org client
    """
    ORG_URL = "https://api.github.com/orgs/{org}"

    def __init__(self, org_name: str) -> None:
        """Init method of GithubOrgClient"""
        self._org_name = org_name

    @memoize
    def org(self) -> Dict:
        """Memoize org"""
        return get_json(self.ORG_URL.format(org=self._org_name))

    @property
    def _public_repos_url(self) -> str:
        """Public repos URL"""
        return self.org["repos_url"]

    @memoize
    def repos_payload(self) -> Dict:
        """Memoize repos payload"""
        return get_json(self._public_repos_url)

    def public_repos(self, license: str = None) -> List[str]:
        """Public repos"""
        json_payload = self.repos_payload
        public_repos = [
            repo["name"] for repo in json_payload
            if license is None or self.has_license(repo, license)
        ]

        return public_repos

    @staticmethod
    def has_license(repo: Dict[str, Dict], license_key: str) -> bool:
        """Static: has_license"""
        assert license_key is not None, "license_key cannot be None"
        try:
            has_license = access_nested_map(repo, ("license", "key")) == license_key
        except KeyError:
            return False
        return has_license
```

- fixtures.py

```sh
#!/usr/bin/env python3

TEST_PAYLOAD = [
  (
    {"repos_url": "https://api.github.com/orgs/google/repos"},
    [
      {
        "id": 7697149,
        "node_id": "MDEwOlJlcG9zaXRvcnk3Njk3MTQ5",
        "name": "episodes.dart",
        "full_name": "google/episodes.dart",
        "private": False,
        "owner": {
          "login": "google",
          "id": 1342004,
          "node_id": "MDEyOk9yZ2FuaXphdGlvbjEzNDIwMDQ=",
          "avatar_url": "https://avatars1.githubusercontent.com/u/1342004?v=4",
          "gravatar_id": "",
          "url": "https://api.github.com/users/google",
          "html_url": "https://github.com/google",
          "followers_url": "https://api.github.com/users/google/followers",
          "following_url": "https://api.github.com/users/google/following{/other_user}",
          "gists_url": "https://api.github.com/users/google/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/google/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/google/subscriptions",
          "organizations_url": "https://api.github.com/users/google/orgs",
          "repos_url": "https://api.github.com/users/google/repos",
          "events_url": "https://api.github.com/users/google/events{/privacy}",
          "received_events_url": "https://api.github.com/users/google/received_events",
          "type": "Organization",
          "site_admin": False
        },
        "html_url": "https://github.com/google/episodes.dart",
        "description": "A framework for timing performance of web apps.",
        "fork": False,
        "url": "https://api.github.com/repos/google/episodes.dart",
        "forks_url": "https://api.github.com/repos/google/episodes.dart/forks",
        "keys_url": "https://api.github.com/repos/google/episodes.dart/keys{/key_id}",
        "collaborators_url": "https://api.github.com/repos/google/episodes.dart/collaborators{/collaborator}",
        "teams_url": "https://api.github.com/repos/google/episodes.dart/teams",
        "hooks_url": "https://api.github.com/repos/google/episodes.dart/hooks",
        "issue_events_url": "https://api.github.com/repos/google/episodes.dart/issues/events{/number}",
        "events_url": "https://api.github.com/repos/google/episodes.dart/events",
        "assignees_url": "https://api.github.com/repos/google/episodes.dart/assignees{/user}",
        "branches_url": "https://api.github.com/repos/google/episodes.dart/branches{/branch}",
        "tags_url": "https://api.github.com/repos/google/episodes.dart/tags",
        "blobs_url": "https://api.github.com/repos/google/episodes.dart/git/blobs{/sha}",
        "git_tags_url": "https://api.github.com/repos/google/episodes.dart/git/tags{/sha}",
        "git_refs_url": "https://api.github.com/repos/google/episodes.dart/git/refs{/sha}",
        "trees_url": "https://api.github.com/repos/google/episodes.dart/git/trees{/sha}",
        "statuses_url": "https://api.github.com/repos/google/episodes.dart/statuses/{sha}",
        "languages_url": "https://api.github.com/repos/google/episodes.dart/languages",
        "stargazers_url": "https://api.github.com/repos/google/episodes.dart/stargazers",
        "contributors_url": "https://api.github.com/repos/google/episodes.dart/contributors",
        "subscribers_url": "https://api.github.com/repos/google/episodes.dart/subscribers",
        "subscription_url": "https://api.github.com/repos/google/episodes.dart/subscription",
        "commits_url": "https://api.github.com/repos/google/episodes.dart/commits{/sha}",
        "git_commits_url": "https://api.github.com/repos/google/episodes.dart/git/commits{/sha}",
        "comments_url": "https://api.github.com/repos/google/episodes.dart/comments{/number}",
        "issue_comment_url": "https://api.github.com/repos/google/episodes.dart/issues/comments{/number}",
        "contents_url": "https://api.github.com/repos/google/episodes.dart/contents/{+path}",
        "compare_url": "https://api.github.com/repos/google/episodes.dart/compare/{base}...{head}",
        "merges_url": "https://api.github.com/repos/google/episodes.dart/merges",
        "archive_url": "https://api.github.com/repos/google/episodes.dart/{archive_format}{/ref}",
        "downloads_url": "https://api.github.com/repos/google/episodes.dart/downloads",
        "issues_url": "https://api.github.com/repos/google/episodes.dart/issues{/number}",
        "pulls_url": "https://api.github.com/repos/google/episodes.dart/pulls{/number}",
        "milestones_url": "https://api.github.com/repos/google/episodes.dart/milestones{/number}",
        "notifications_url": "https://api.github.com/repos/google/episodes.dart/notifications{?since,all,participating}",
        "labels_url": "https://api.github.com/repos/google/episodes.dart/labels{/name}",
        "releases_url": "https://api.github.com/repos/google/episodes.dart/releases{/id}",
        "deployments_url": "https://api.github.com/repos/google/episodes.dart/deployments",
        "created_at": "2013-01-19T00:31:37Z",
        "updated_at": "2019-09-23T11:53:58Z",
        "pushed_at": "2014-10-09T21:39:33Z",
        "git_url": "git://github.com/google/episodes.dart.git",
        "ssh_url": "git@github.com:google/episodes.dart.git",
        "clone_url": "https://github.com/google/episodes.dart.git",
        "svn_url": "https://github.com/google/episodes.dart",
        "homepage": None,
        "size": 191,
        "stargazers_count": 12,
        "watchers_count": 12,
        "language": "Dart",
        "has_issues": True,
        "has_projects": True,
        "has_downloads": True,
        "has_wiki": True,
        "has_pages": False,
        "forks_count": 22,
        "mirror_url": None,
        "archived": False,
        "disabled": False,
        "open_issues_count": 0,
        "license": {
          "key": "bsd-3-clause",
          "name": "BSD 3-Clause \"New\" or \"Revised\" License",
          "spdx_id": "BSD-3-Clause",
          "url": "https://api.github.com/licenses/bsd-3-clause",
          "node_id": "MDc6TGljZW5zZTU="
        },
        "forks": 22,
        "open_issues": 0,
        "watchers": 12,
        "default_branch": "master",
        "permissions": {
          "admin": False,
          "push": False,
          "pull": True
        }
      },
      {
        "id": 7776515,
        "node_id": "MDEwOlJlcG9zaXRvcnk3Nzc2NTE1",
        "name": "cpp-netlib",
        "full_name": "google/cpp-netlib",
        "private": False,
        "owner": {
          "login": "google",
          "id": 1342004,
          "node_id": "MDEyOk9yZ2FuaXphdGlvbjEzNDIwMDQ=",
          "avatar_url": "https://avatars1.githubusercontent.com/u/1342004?v=4",
          "gravatar_id": "",
          "url": "https://api.github.com/users/google",
          "html_url": "https://github.com/google",
          "followers_url": "https://api.github.com/users/google/followers",
          "following_url": "https://api.github.com/users/google/following{/other_user}",
          "gists_url": "https://api.github.com/users/google/gists{/gist_id}",
          "starred_url": "https://api.github.com/users/google/starred{/owner}{/repo}",
          "subscriptions_url": "https://api.github.com/users/google/subscriptions",
          "organizations_url": "https://api.github.com/users/google/orgs",
          "repos_url": "https://api.github.com/users/google/repos",
          "events_url": "https://api.github.com/users/google/events{/privacy}",
          "received_events_url": "https://api.github.com/users/google/received_events",
          "type": "Organization",
          "site_admin": False
        },
        "html_url": "https://github.com/google/cpp-netlib",
        "description": "The C++ Network Library Project -- header-only, cross-platform, standards compliant networking library.",
        "fork": True,
        "url": "https://api.github.com/repos/google/cpp-netlib",
        "forks_url": "https://api.github.com/repos/google/cpp-netlib/forks",
        "keys_url": "https://api.github.com/repos/google/cpp-netlib/keys{/key_id}",
        "collaborators_url": "https://api.github.com/repos/google/cpp-netlib/collaborators{/collaborator}",
        "teams_url": "https://api.github.com/repos/google/cpp-netlib/teams",
        "hooks_url": "https://api.github.com/repos/google/cpp-netlib/hooks",
        "issue_events_url": "https://api.github.com/repos/google/cpp-netlib/issues/events{/number}",
        "events_url": "https://api.github.com/repos/google/cpp-netlib/events",
        "assignees_url": "https://api.github.com/repos/google/cpp-netlib/assignees{/user}",
        "branches_url": "https://api.github.com/repos/google/cpp-netlib/branches{/branch}",
        "tags_url": "https://api.github.com/repos/google/cpp-netlib/tags",
        "blobs_url": "https://api.github.com/repos/google/cpp-netlib/git/blobs{/sha}",
        "git_tags_url": "https://api.github.com/repos/google/cpp-netlib/git/tags{/sha}",
        "git_refs_url": "https://api.github.com/repos/google/cpp-netlib/git/refs{/sha}",
        "trees_url": "https://api.github.com/repos/google/cpp-netlib/git/trees{/sha}",
        "statuses_url": "https://api.github.com/repos/google/cpp-netlib/statuses/{sha}",
        "languages_url": "https://api.github.com/repos/google/cpp-netlib/languages",
        "stargazers_url": "https://api.github.com/repos/google/cpp-netlib/stargazers",
        "contributors_url": "https://api.github.com/repos/google/cpp-netlib/contributors",
        "subscribers_url": "https://api.github.com/repos/google/cpp-netlib/subscribers",
        "subscription_url": "https://api.github.com/repos/google/cpp-netlib/subscription",
        "commits_url": "https://api.github.com/repos/google/cpp-netlib/commits{/sha}",
        "git_commits_url": "https://api.github.com/repos/google/cpp-netlib/git/commits{/sha}",
        "comments_url": "https://api.github.com/repos/google/cpp-netlib/comments{/number}",
        "issue_comment_url": "https://api.github.com/repos/google/cpp-netlib/issues/comments{/number}",
        "contents_url": "https://api.github.com/repos/google/cpp-netlib/contents/{+path}",
        "compare_url": "https://api.github.com/repos/google/cpp-netlib/compare/{base}...{head}",
        "merges_url": "https://api.github.com/repos/google/cpp-netlib/merges",
        "archive_url": "https://api.github.com/repos/google/cpp-netlib/{archive_format}{/ref}",
        "downloads_url": "https://api.github.com/repos/google/cpp-netlib/downloads",
        "issues_url": "https://api.github.com/repos/google/cpp-netlib/issues{/number}",
        "pulls_url": "https://api.github.com/repos/google/cpp-netlib/pulls{/number}",
        "milestones_url": "https://api.github.com/repos/google/cpp-netlib/milestones{/number}",
        "notifications_url": "https://api.github.com/repos/google/cpp-netlib/notifications{?since,all,participating}",
        "labels_url": "https://api.github.com/repos/google/cpp-netlib/labels{/name}",
        "releases_url": "https://api.github.com/repos/google/cpp-netlib/releases{/id}",
        "deployments_url": "https://api.github.com/repos/google/cpp-netlib/deployments",
        "created_at": "2013-01-23T14:45:32Z",
        "updated_at": "2019-11-15T02:26:31Z",
        "pushed_at": "2018-12-05T17:42:29Z",
        "git_url": "git://github.com/google/cpp-netlib.git",
        "ssh_url": "git@github.com:google/cpp-netlib.git",
        "clone_url": "https://github.com/google/cpp-netlib.git",
        "svn_url": "https://github.com/google/cpp-netlib",
        "homepage": "http://cpp-netlib.github.com/",
        "size": 8937,
        "stargazers_count": 292,
        "watchers_count": 292,
        "language": "C++",
        "has_issues": False,
        "has_projects": True,
        "has_downloads": True,
