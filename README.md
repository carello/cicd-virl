# CI/CD using VIRL, DRONE and GOGS


### Overview

This application is a CI/CD case study using Cisco VIRL simulator, DRONE and GOGS. Images are pulled from hub.docker.com. This case study doesn't cover installing or configuring VIRL.  There are many ways to set up management access into VIRL. I set up a Shared Flat Network toplogy to allow Ansible access to the virtual devices within VIRL. This can be accomplished through simple routing or using an OpenVPN connection. Documentation can ge found at [here](http://virl.cisco.com). This app was developed on MAC OS X.

### Software (prerequisites)
You'll need to following software installed:

*	[Docker for Desktop](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac)
* Git. There are a couple of installation option/choices:
	* 	[Git using Homebrew](https://www.atlassian.com/git/tutorials/install-git)
	* [Git software installer](https://git-scm.com/downloads)
* [Drone CLI](https://docs.drone.io/cli/install/)
* [Docker Hub](docker.io) account.
* VIRL is available via subscription or there's free access on Cisco DevNet. However this app was developed on a dedicated VIRL server. I haven't tested this app on the hosted VIRL on Cisco DevNet.
	* [VIRL](http://virl.cisco.com)
	* [Cisco DeveNet](http://developer.cisco.com)
	* [Cisco CICD lab](https://developer.cisco.com/learning/modules/netcicd)



## How to Use
### Section 1: Bootstrapping

* Open a terminal:
	* clone this repo
	* `cd cicd-virl`
	* `cd cicd-virl-prep`
	* Enter `source ./drone-env.sh` to load environmental variables.
	* Enter `./startup.sh` to start up Drone and Gogs

### Section 2: Set up GOGS

- Open a browser to `http://localhost:3000` (port 3000 is defined in the yaml file).
	* Use SQLite3
	* Set the Domain to: `dc-gogs`
	* SSH port: clear this field (blank).
	* Set 'Application URL' to: dc-gogs and port 3000 . ie,` http://dc-gogs:3000`
	* Create an admin user
	* Hit submit. Note that the browser might not refresh. Just open another browser to `http://localhost:3000` , login and create a repo called: `network_cicd_lab`. You must name this exactly as indicated. 

- Open another terminail window and navigate into your cloned repo `cicd-virl`
	* `cd cicd-virl-data`
	* `cd network_cicd_lab`
	* Enter the following commands:
		* `git init`
		* `git add .`
		* `git commit -m "first commit"` 	
		* `git remote add origin http://localhost:3000/<repo-name>/network_cicd_lab.git`
		* `git push -u origin master`
		* `git branch dev`
		* `git checkout dev`
		* `git push -u origin dev`
	* Check your GOGS repo, you should now have two branches: master and dev


### Section 3: Set up Webex Teams (Optional)

- In a new broswer window, go to: `https://developer.webex.com` and login. (If you don't have an account, sign up for a free account).
- Click on your avatar in the upper right corner and select "My Webex Teams Apps"
- Click "Create a New App"
- Click "Create a Bot"
- Create your bot with the following information:
	* Name: CICD BOT
	* Bot Username: Enter any avaiable name, (indicated in green when you enter a name).
	* Icon: Select a default choice
	* Description: CICD-VIRL Bot
	* Click: "Add Bot"
	* A new screen will appear. Click the button to copy the "Bot's Access Token"
		* We will need the Bot's Access Token in the next section; save it somewhere temporarily. 



### Section 4: Set up Drone 

- Open another browser to `http://localhost`and enter your user info you created earlier in the Gogs setup. Wait for DRONE to sync. __Do not activate the repo yet__.

- We need to grab a few Drone environmental variables. Go to `User Settings` (upper right ICON) and copy the DRONE\_SERVER and DRONE\_TOKEN variables. It looks something like this:

		export DRONE_SERVER=http://localhost
		export DRONE_TOKEN=u4C5ghLKVTdafadjfldjfladfjaFDRs


- Go back to your open terminal and paste these in. Run `drone info` to check if things are working.

- We'll need to setup several secret keys next. We'll use the DRONE GUI to set up the keys but this can also be accomplished using CLI. The keys we'll be configuring are:
	* VIRL_HOST="VIRL IP Address"
	* VIRL_USERNAME=guest (NOTE: this is the default username in VIRL)
	* VIRL_PASSWORD=guest (NOTE: this is the default password in VIRL)
	* GOGS_USER="Account name you used to set up GOGs earlier"
	* GOGS_PASS="Account password you used to set up GOGs earlier"
	* DOCKER_USERNAME="Your Docker Hub username"
	* DOCKER_PASSWORD="Your Docker Hub password"
	* SPARK_TOKEN="Access Token you copied from Section 3 above)
	* PERSONEMAIL="Email address from you Webex Teams account"  


- Select `Activate` repository in Drone
	* Check the `Trusted` in the Project Settings. (Don't hit SAVE yet)
	* Scroll down to the Secrets section and enter the keys. For example enter: 
		* `VIRL_HOST` in the Secret Name field
		* `[IP Address of the VIRL host]` in the Secret Value field.
		* Click: Add a Secret 

	* Repeat these setps for all the keys mentioned above
	* Select SAVE when done entering all the keys
	* Select Repositories (located on the top of the page) to bring you back to your repo

- Go to your Gogs browser and into your repo, (you should already at this location).
	* On the right hand side select the `settings` link.
	* On the left hand side, select the tab for `webhooks`.
	* You should then see a link in the center of the page for `http://dc-drone-server/hook`, select it, and scroll towards the bottom of the page; there is a button for `Test Delivery`. If things are correct you should make a successful connection, (scroll down the page to see the result).

### Section 5: CI/CD in Action

- Go back to your 2nd terminal window. You should be in the `network_cicd_lab` directory.
- Enter `git branch -a` to verify which branch you are currently in.
- Enter `git checkout dev` to switch to the dev branch.
- We need to make a change to kick off the workflow. A simple way to do this is create a empty file. Enter the following commands:
	* `touch dummy.txt`
	* `git add .`
	* `git commit -m "2nd commit on DEV"` (or whatever message you'd like)
	* `git push`
- Go back to the Drone browser and watch your workflow execute. It will take about 3-4 mins to complete.

### Section 6: Cleanup

- Go back to your 1st terminal window. You should be in the `cics-virl-prep` directory.
- enter: `./cleanup.sh`

### To Do
- Create a robust/flexible app within the drone plugins and not rely on 'commands' within the .drone.yml file. This would allow for more flexible use cases in the future.
- Webex BOT isn't working accurately.
