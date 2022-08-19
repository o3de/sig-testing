# Github Codeowners File

## Summary

O3DE currently does not have code ownership automation. This means that processes such as creating pull requests and GitHub issues requires manually adding the correct Special Interest Group (SIG). For a contributor who is not familiar with the code base this can be an intimidating task. The user will have to either scour documentation or ask and wait for responses in the O3DE discord chats. Not only is this process is tedious, but can potentially lead towards incorrect code owner reviews on pull requests.

This RFC will discuss utilizing GitHub Codeowners as a solution, along with answering implementation details.

## What is the relevance of this feature?

Codeowners is a well known public tool that adds automation capabilities for code ownership in a GitHub repository. This will label directories and files to their corresponding SIG. The Codeowners file will also label unowned code, which can be used ensure that each file in O3DE has an owner. It's important that the tool is well known to the open source community since O3DE is also open source.

## Feature design description:

A Codeowners file will be created in the O3DE repository. The file will contain manually added mappings between directories and GitHub users, which can include a SIG team alias. The file should be separated by SIG's, which will organize their own directory ownerships. Having a GitHub Codeowners file in the O3DE repository will automatically add all corresponding SIG's to for each file affected by a Pull Request. Viewing a file in the O3DE repository on the GitHub website will also indicate who the owning SIG is.

There are multiple CLI tools that contributors can take advantage of. These tools can quickly tell a user the owner of a specified file without having to open a Pull Request or look on GitHub. These tools can also scan the repository for all unowned files which can be utilized to ensure every line of code has an owner.

## Technical design description:

Setting up the GitHub Codeowners file should follow the [official documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners).

A file will be created at ~/o3de/.github/CODEOWNERS as per the documentation above. The file's content will be grouped by SIG's who will be responsible for specifying their own areas of ownership. Once the file is added to the repository, its effects will automatically take place.

```
# SIG-Core
Code\Editor @sig-core
Code\Foo @sig-core @sig-testing  # Example of Co-Owned code

# SIG-Testing
Tools\LyTestTools @sig-testing
```

The Codeowners file will cease to function if the size exceeds 3MB. The [Early Warning System](https://docs.o3de.org/docs/contributing/to-code/validation-errors/) should add a new validator to ensure the size of the file is below the threshold.

### Un-owned Code Policy

Initially, we should not enforce that newly added files to the O3DE repository have an owner because it will require some effort and coordination to reach a point where all files/directories have an owner. Eventually, the Early Warning System should block Pull Requests that adds new files with no owner added in the Codeowners file. This feature can be implemented as part of this initiative, but should only be enabled once the codebase already has an owner for all files.

### GitHub Codeowners Ownership

The ownership of the GitHub Codeowners file itself should be owned or determined by the Technical Steering Committee as it's an important organizational key to the repository. It has been suggested that SIG-Ops could be a potential owner as the group is responsible for automation.

## What are the advantages of the feature?

GitHub Codeowners automatically adds the designated owners of each file in a pull request, ensuring that the correct SIG reviews all changes on files they own. There are CLI tools that users can use to determine which SIG owns a file. This will reduce the amount of triaging for O3DE GitHub Issues with no SIG label. Most importantly, the Codeowners file can enable further types of automation such as targeted SIG notifications on Automated Review failures.

## What are the disadvantages of the feature?

Automation can produce an overwhelming amount of noise and notifications. While this shouldn't be an issue during the initial implementation of the Codeowners file, it's important that any future automation keeps this in mind.

## Are there any alternatives to this feature?

O3DE can implement its own custom ownership system by utilizing CMake. This would allow for full customization of code ownership automation for O3DE. This path is not suggested due to the implementation effort difference between a custom system and the GitHub Codeowners file. It is also important to use a well known public tool as previously mentioned above for public O3DE contributors.

## How will users learn this feature?

Since the process is automatic, there is not a large amount of teaching required. An email and discord notification announcing the implementation and linking the official documentation should suffice.

## Are there any open questions?

After the feature rollout, there should be an effort to establish ownership for all unowned code. Once all unowned is accounted for, then the new file code ownership validator should be enabled.
