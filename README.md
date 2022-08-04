# bump-tags
## _A small project which will be bumping up the tags based on new releases._

![Build status](https://github.com/nikeniks/bump-tags/actions/workflows/publish.yml/badge.svg)

## 🏆 Feature
- Determine what the next tag version will be
- Using Maven, build and publish your application to GitHub packages
- Create a GitHub release
- Create the next tag version

## :man_technologist: Tech
- Java spring boot  
- Maven
- https://github.com/anothrNick/github-tag-action to determine and create next tag version
- https://github.com/elgohr/Github-Release-Action to create new Github release
- Github for version control
- Github Actions for build pipeline

## :gear: Working

Working is quite simple for this application.

- The build checks for your code and based on the commit message it will determine the next tag version.
- You can use either #major, #minor, #patch, or #none in your commit message to specify which tag version will be updated.
- If no #major, #minor or #patch is provided in commit message then "DEFAULT_BUMP" configuration settings is selected for next tag version, by default the "DEFAULT_BUMP" is set tp #minor.
- For #none in commint message the pipeline only runs "generate-tags" step in build pipeline thats because I have added a condition `if needs.generate-tags.outputs.new_tag!=needs.generate-tags.outputs.tag` if this condtion fails the subsequent jobs are skipped.
- This is done so that the `mvn deploy` does not fail while publishing same version to github packages. 
```
transfer failed for https://maven.pkg.github.com/nikeniks/bump-tags/com/example/demo/0.0.0/demo-0.0.0.jar, status: 409 Conflict
```

## :bug: Bug in https://github.com/anothrNick/github-tag-action
- While working on the action provided by `anothrNick/github-tag-action` I have found a possible bug. I am not completely sure but it certainly looks like it.
- When you use `#none` in the commit message the action performs as expected and does not determine the next tag.
- But for the next commit which is without `#none` in its message the action still considers that the commit message contains `#none` and does not determine next tag.
- This behaviour is seen even though you set `DEFAULT_BUMP` to minor.

### :microscope: Test results for above bug observation
1) https://github.com/nikeniks/bump-tags/runs/7656311775?check_suite_focus=true

`#none test changes` is commit message.
- Expected Behaviour: Ignore `DEFAULT_BUMP` setting and skip determining next tag.
- Resultant Behaviour: As expected 
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
2) https://github.com/nikeniks/bump-tags/runs/7656330681?check_suite_focus=true

`checking if minor update takes palce` is commit message but even then it was skip.
- Expected behaviour:  Should have been to detemine next tagbased on `DEFAULT_BUMP` settings, 
- Resultant Behaviour: Check the message below, the commit message that action received was 
		    `checking if minor update takes palce #none test changes` that is basically commit message from current commit + commit message from previous 			commit. And becase of that it checked for #none and skipped the next tag.
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
checking if minor update takes palce #none test changes
Default bump was set to none. Skipping...
```
3) https://github.com/nikeniks/bump-tags/actions/runs/2790949088
To overcome this behaviour you will need to add #minor, #major or #patch in the next commit.

`#minor because DEFAULT_BUMP doesnt work. There is a issue with DEFAULT_BUMP, stops working when you use #none in commit message and all the subsequent commits even without #none are considered with #none in commit message. checking if minor tag is updates. checking if minor update takes palce #none test changes` commit message.
- Expected Behaviour: Determines the next tag based on commit message and bump it up.
- Resultant Behaviour: As expected.

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
#minor because DEFAULT_BUMP doesnt work. There is a issue with DEFAULT_BUMP, stops working when you use #none in commit message and all the subsequent commits even without #none are considered with #none in commit message. checking if minor tag is updates. checking if minor update takes palce #none test changes
minor
Bumping tag v0.10.0. 
	New tag v0.11.0
```

### After the above step you dont need to specify the manual bump. It again starts accepting the automatic bump that is mentioned in `DEFAULT_BUMP`. 

