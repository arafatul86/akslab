# LAB 0: Initial Setup

## Work Environment

There are two environments you will be working in for the exercises today.

1. **Jumpbox:** The apps and containers must be run on a Linux machine. 

    > Note: If you have bash or ssh available on your machine, it is easiest to access the jump box via SSH. Otherwise, RDP is required.

2. **Azure Cloud Shell:** The Azure Cloud Shell will be accessed by logging into the Azure Portal (http://portal.azure.com).

Labs 1 and 2 require the Jumpbox. The subsequent labs all use the Azure Cloud Shell.

## Setup the Jumpbox VM

You need to have Azure Command Line Interface(CLI) or Powershell and Git installed on you machine in order to be able complete the manual setup.

<a href="https://azure.microsoft.com/en-us/downloads/"/>Azure tools</a>

<a href="https://git-scm.com/downloads"/>Git download</a>

## Easy setup

1. Start the jumpbox deployment by clicking on the 'Deploy to Azure' button below
 - Note down the adminUsername and adminPassword parameters, you will need them on step 3.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fncmpuolitaival%2Fakslab%2Fmaster%2Fresources%2Faks-jumpvm.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

2. To get the jumpbox FQDN you need to go to the 'akslab-jumpbox' resource group in Azure Portal, select the Public IP resource and copy the DNS name

3. Login to the jumpbox with SSH

## Manual setup:

1. Clone the Github repo via the command line

    ```
    git clone https://github.com/ncmpuolitaival/akslab.git
    ```
2. Create new Resource Group
    ```
    AZ CLI: az group create --name akslab-jumpbox --location westeurope
    ```
    Or
    ```
    Powershell: New-AzureRmResourceGroup -Name akslab-jumpbox -Location westeurope
    ```
3. Deploy the jumpbox to the Resource Group you created in the previous step
    ```
    cd akslab/resources
    ```
    And then
    ```
    AZ CLI: az group deployment create --name akslabdeploy --resource-group akslab-jumpbox --template-file ./aks-jumpvm.json --parameters ./aks-jumpvm.parameters.json
    ```
    Or
    ```
    Powershell: New-AzureRmResourceGroupDeployment -Name akslabdeploy -ResourceGroupName akslab-jumpbox -TemplateFile aks-jumpvm.json -TemplateParameterFile aks-jumpvm.parameters.json
    ```
4. Jumpbox FQDN, username and password are listed in the deployment output

5. Login to the jumpbox via SSH

# LAB 1: Running App on Local Machine

## Clone Lab Github Repo

Once you have accessed the jumpbox, you must clone the workshop repo to the machine.

1. Start with a terminal on the jumpbox
2. Clone the Github repo via the command line

    ```
    git clone https://github.com/Azure/blackbelt-aks-hackfest.git
    ```

## Get Applications up and running

### Database layer - MongoDB

The underlying data store for the app is [MongoDB](https://www.mongodb.com/ "MongoDB Homepage"). It is already running. We need to import the data for our application.

1. Import the data using a terminal session on the jumpbox

    ```bash
    cd ~/blackbelt-aks-hackfest/app/db/

    mongoimport --host localhost:27017 --db webratings --collection heroes --type json --file ./heroes.json --jsonArray && mongoimport --host localhost:27017 --db webratings --collection ratings --type json --file ./ratings.json --jsonArray && mongoimport --host localhost:27017 --db webratings --collection sites --type json --file ./sites.json --jsonArray
    ```

### API Application layer - Node.js

The API for the app is written in javascript, running on [Node.js](https://nodejs.org/en/ "Node.js Homepage") and [Express](http://expressjs.com/ "Express Homepage")

1. Go to

    ```bash
    cd ~/blackbelt-aks-hackfest/app/api
    ```
2. Modify package.json file with preferred editor, change port number on row number 8 from 27019 to 27017
3. Update dependencies and run app via node in a terminal session on the jumpbox

    ```bash
    npm install && npm run localmachine
    ```

3. Open a new terminal session on the jumpbox and test the API

    use curl
    ```bash
    curl http://localhost:3000/api/heroes
    ```
    If you are in an RDP session, you can browse to <http://localhost:3000/api/heroes>

### Web Application layer - Vue.js, Node.js

The web frontend for the app is written in [Vue.js](https://vuejs.org/Vue "Vue.js Homepage"), running on [Node.js](https://nodejs.org/en/ "Node.js Homepage") with [Webpack](https://webpack.js.org/ "Webpack Homepage")

1. Open a new terminal session on the jumpbox
2. Update dependencies and run app via node

    ```bash
    cd ~/blackbelt-aks-hackfest/app/web

    npm install && npm run localmachine
    ```
3. Test the web front-end

    The jumpbox has an external DNS name and port 8080 is open. You can browse your running app with a link such as: http://akslab-jumpvm-j6efwsdqztpsm.westeurope.cloudapp.azure.com:8080 

    You can also test from a new terminal session in the jumpbox
    ```bash
    curl http://localhost:8080
    ```

## Clean-up

Close the web and api apps in the terminal windows by hitting `ctrl-c` in each of the corresponding terminal windows

Stop the MongoDB to free the port 27017 on the jumpbox.
    ```bash
    sudo systemctl stop mongod
    ```


## Contiune labs

Proceed with other labs on  https://github.com/Azure/blackbelt-aks-hackfest

Direct link to next lab: https://github.com/Azure/blackbelt-aks-hackfest/blob/master/labs/day1-labs/02-dockerize-apps.md
