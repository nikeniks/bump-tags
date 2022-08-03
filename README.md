# bump-tags
## _A small project which will be bumping up the tags based on new releases._

## Feature
- Determine what the next tag version will be.
- Using Maven, build and publish your application to GitHub packages.
- Create a GitHub release.
- Create the next tag version.

## Tech
- Java spring boot  
- Maven
- https://github.com/anothrNick/github-tag-action to determine and create next tag version.
- https://github.com/elgohr/Github-Release-Action to create new Github release.
- Github for version control
- Github Actions for build pipeline

## Working

Working is quite simple for this application.

- The build checks for your code and based on the commit message it will determine the next tag version.
- You can use either #major, #minor, #patch, or #none in your commit message to specify which tag version will be updated
- If no #major, #minor or #patch is provided in commit message then "DFAULT_BUMP" configuration settings is selected for next tag version, by default the "DEFAULT_BUMP" is set tp #minor
- For #none in commint message the pipeline only runs "generate-tags" step in build pipeline thats because I have added a condition `if needs.generate-tags.outputs.new_tag!=needs.generate-tags.outputs.tag` if this condtion fails the subsequent jobs are skipped.
- This is done so that the `mvn deploy` does not fail while publishing same version to github packages. 
```
transfer failed for https://maven.pkg.github.com/nikeniks/bump-tags/com/example/demo/0.0.0/demo-0.0.0.jar, status: 409 Conflict
```

## Bug in https://github.com/anothrNick/github-tag-action
- While working on the action provided by `anothrNick/github-tag-action` I have found a possible bug. I am not completely sure but it certainly looks like it.
- When you use `#none` in the commit message the action performs as expected and does not determain the next tag.
- But for the next commit which is without `#none` in its message the action still considers that the commit message contains `#none` and does not determine next tag.
- This behaviour is seen even though you set `DEFAULT_BUMP` to minor.

### Test results for above bug observation
1) https://github.com/nikeniks/bump-tags/runs/7656311775?check_suite_focus=true
`#none test changes` is commit message
```
*** CONFIGURATION ***
	DEFAULT_BUMP: minor
	WITH_V: true
	RELEASE_BRANCHES: master,main
	CUSTOM_TAG: 
	SOURCE: .
	DRY_RUN: true
	INITIAL_VERSION: 0.0.0
	TAG_CONTEXT: repo
	PRERELEASE_SUFFIX: beta
	VERBOSE: true
Is master a match for main
Is main a match for main
pre_release = false
#none test changes
Default bump was set to none. Skipping...
```


