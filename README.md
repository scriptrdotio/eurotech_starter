# eurotech_starter

- [Overview](./REAMDE.md#overview)
- [Installing and configuring the application](./README.md#installing-and-configuring-the-application)
- [Running the application](./README.md#running-the-application)

## Overview

This is a very simple application consisting of a simple dashboard that monitors metrics sent by simulated Eurotech devices. The latter publish data to a eurotech Everyware mqtt topic. Two types of events are published:

- location and speed data, published while the bus is moving
- bus load data (passengers getting off, getting on, current number of passengers), published while the bus is stopped.

On the scriptr side, a script is subscribed to the Everyware mqtt topic. As soon as events arrive, they are are persisted in the scriptr's data store and further published to the dashboard that is updated in real time.

## Pre-requisites

You need an Everyware account:

- [Everyware account](https://console-sandbox.everyware-cloud.com/) (sandbox in this example) and you need to create an mqtt topic in the platform to which the simulator will publish data
- A [Github account](https://github.com/)

# Installing and configuring the application

## Configure Github in your scriptr account

To proceed with the installation steps, you need to have a Github repository and a Github Personal Access Token. If you don' t have any of them, check the below links:

- [Greate a new Github repository](https://help.github.com/articles/create-a-repo/)
- [Generate a personal access token from Github](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)

To configure the Github settings in scriptr, open your [scriptr workspace](https://www.scriptr.io/workspace), click on your username in the top right corner of the screen, then select **Settings** then click on the **Github** tab. 

Fill the fields as follows:
- Repository Owner: scriptrdotio
- Access Token: your personal access token or the one provided above 
- Repository Name: demo
- Branch: master
- Path: leave empty

Click "Save" to validate you changes.

## Import the source code from Github

From your [scriptr workspace](https://www.scriptr.io/workspace), click on the arrow next to +New Script in the bottom left corner of the screen:
- Click on **Install Modules**
- In the Modules dialog:
  - Expand **+Add Custom Module from GitHub** 
  - Use scriptrdotio as account owner
  - Use eurotech_starter as Repository name
  - Set the path to "/src" to only checkout the source code (and not the documentation)
  - Use /eurotech_starter as destination folder
  - Click Install when done

## Configure the application

### Create a sub-domain for your scriptr account

If you do not already have a sub-domain: from your [scriptr workspace](https://www.scriptr.io/workspace), click on your username then select **Account**
- Select the **Sub-domain** tab
- Enter a name for your sub-domain (it has to be unique, scriptr will reject names that already exist)
- click on close

### Create channels

Channels are used by scriptr as abstractions of publish/subscribe mechanisms. We will create two channels:

- The **eurotech** channel will be used to convey any message received on the Everyware topic we are subscribed to, to our application
- The **responseChannel** channel will be used to publish data to our dashboard in real-time

From your [scriptr workspace](https://www.scriptr.io/workspace), click on your username in the top right corner of the screen:
- Click on Settings then select the **Channels** tab
- In the dialog, expand +Add Channel
- Enter a name for your channel (eurotech)
- Click on the checkbox to the right to validate

Proceed similarly to create the second channel (responseChannel) but this time, check the **Allow anonymous subscription** checkbox.

### Create a device

You need to create a device in scriptr that is the representation of the physical device. Therefore, from the [scriptr workspace](https://www.scriptr.io/workspace):

- Click on your username in the top right corner of the screen
- Click on **Device directory**
- Click **+Add Device**
- Enter the same value for the id and name fields. **Make sure that they match the id of a device you've created using the [Eurotech simulator](https://cs.eurotech.com/gps-pcn-simulator/)** or an actual device id in your Everyware account
- Enter some password, then click on the checkbox to save your changes
- **Copy the authentication token that is associated to the device, as you will need it in the [create a bridge](./README.md#create-a-bridge) step**

### Subscribe to the Everyware MQTT topic

The devices used in this application publish data to two distinct topics hosted by the Everyware platform:

- {account}/{client_id}/PCNPublisher/LocationPublisher/location. Published data are position_speed, position_longitude, position_latitude
- {account}/{client_id}/PCNPublisher/Bus. Published data are position_longitude, position_latitude, AbsolutePop, AbsoluteOut, AbsoluteIn 

To automatically convey these messages to your scriptr account, you need to create an endpoint + bridge on scriptr that subscribe to the above. 

#### Create an endpoint 

From the [scriptr workspace](https://www.scriptr.io/workspace):

- Click on your username in the top right corner of the screen:
- Click on Settings then select the **External Endpoints** tab
- Click on **+Add External Endpoint**
- From the **Type** drop-down field, scroll to select **Eurotech**
- Set the value of the **URL** field to that of your Everyware account (e.g. mqtt://broker-sandbox.everyware-cloud.com)
- Set the **Port** field to 1883
- Set the **Username** to *your Everyware username*
- Set the **Password** to *your Everyware password*
- Set the **Topic** to *your_everyware_topic*/your_device_id/+/#, e.g. : Scripr-io/scriptr-bus-001/+/#
- Click on the checkbox button to validate your changes

#### Create a bridge

A bridge connects an endpoint to one of your scriptr channels. Thus to create a bridge, you need to choose a channel:

- Click on your username in the top right corner of the screen
- Click on **Settings** then select the **Channels** tab
- Click on the globe icon near the **eurotech** channel
- From the drop down list, select the **eurotech** endpoint
- Paste your device authentication token (obtained in [create a device](./README.md#create-a-device))
- Click on **Add Bridge** to deploy a new bridge

### Subscribe the processdata script to the eurotech channel

The above configuration subscribes your scriptr account to your mqtt topic on Everyware. All messages that are published to the latter will automatically be received by your **eurotech** channel. In order to start working on the payload contained in these messages, you just need to subscribe a script to the **eurotech** channel:

- Open the **/eurotech_simpleapp/api/inject** script by expanding the code tree on the left of your [workspace](https://www.scriptr.io/workspace)
- In the tool bar click on **Subscribe**
- In the channels list, switch the toggle on for **eurotech**
- Click on Close

# Running the application

## Start the Eurotech simulator

To start generating data using Eurotech's PCN transport simulator, follow the steps described in [Eurotech's documentation](https://github.com/eurotech/pcn-trans-demo/blob/master/docs/web-pcn-sim.md). The below is a summary of what you need to do:

Frist, sign-in to the [simulator](https://cs.eurotech.com/gps-pcn-simulator/) **using your Everyware credentials** (account name, username and password). Make sure to use the credentials of an Everywhere user who is authorized to publish to the mqtt topics of the platform.

### Createa device (optonioal

- If necessary, click on **create device** to create a new simulated device
- Fill the form by entering any name for the device, add a start and end terminals (bus stations) 
- Click on **Create device**

### Enter simulation data

- Enter some data for the simulator. Best option is to copy/paste the [data below](./README.md#simulation-data) (obtained from  [Eurotech's documentation](https://github.com/eurotech/pcn-trans-demo/blob/master/docs/web-pcn-sim.md))
- If you wish, configure the frequency at which messages will be published to Everywhere, as well as the max number of passenger, etc.
- Once ready, click on **Start sending messages**

## Open the dashboard in the browser

Once data starts being generated, it will automatically flow into your scriptr account (provided you configured it already) and be reflected in real time in the dashboard:

- From your [scriptr workspace](https://www.scriptr.io/workspace), expand the code tree on your left and open the **/eurotech_starter/dashboard** script
- Click on **View** in the toolbar to open the dashboard in the browser

# Simulation Data

Copy/paste the below into the simulator

```
27.986519,-82.733574,pcn
27.961201,-82.762928
27.938304,-82.746792
27.938304,-82.750397,pcn,paxin=2,paxout=1
27.938304,-82.75692
27.938304,-82.762585
27.943612,-82.763271,pcn,paxin=1,paxout=2
27.948919,-82.7631
27.954378,-82.762756
27.959837,-82.762413,pcn
27.96393,-82.762756
27.967418,-82.7631
27.972269,-82.763443,pcn
27.977878,-82.763443
27.984852,-82.763443,pcn
27.991825,-82.763443,pcn
27.997433,-82.763443
28.001374,-82.762928,pcn,paxin=5,paxout=1
28.004708,-82.762756
28.008194,-82.762585
28.01168,-82.762756,pcn
28.013347,-82.759151
28.016681,-82.75898,pcn
28.019712,-82.75898
28.019257,-82.754345,pcn
28.019105,-82.750053
28.015771,-82.747135
28.012437,-82.746277,pcn
28.009406,-82.744217
28.006072,-82.742157,pcn
28.001525,-82.742157
27.996372,-82.742157
27.991218,-82.742157,pcn
27.989854,-82.747307
27.989399,-82.752113,pcn
27.989551,-82.75486
27.992886,-82.755032,pcn
27.994098,-82.751083,pcn
```
