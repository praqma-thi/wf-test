# Worfklow test

This project contains a Jenkinsfile showinf off a simple workflow which runs builds differently based on branch names.

* builds for branches matching `ready/*` will be:
	* merged into `master`
	* built
	* published as the new `master`
	* deleted
* builds for branches matching `master` or `hotfix/` will be:
	* built
	* deployed
* builds for branches _not_ matching any of the above will be:
	* merged into `master`
	* built

	
