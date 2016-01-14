# stocks-dashboard-lab
This lab teaches you how to create a realtime dashboard of stock prices using Hortonworks Data Platform and NiFi

# Creating a relatime stock price dashboard using HDP and HDF


## Overview

This lab will teach you how to use Hortonworks Data Platform and NiFi to monitor stock prices and display them on a dashboard. We will go through the following steps in order:

* download and install HDP 2.3 Sandbox
* install NiFi and Solr on the HDP Sandbox
* use NiFi to query stock prices of AAPL, HDP, etc. from Google Finance by making a HTTP call
* do text processing to extract price on the returned HTTP response, in NiFi 
* store the stock symbol and price in HBase and Solr
* create a Banana dashboard to query Solr and display the stock prices 

This is a typical Internet of Anything (IoAT) end-to-end application. The goal is to get you familiar with the components of HDP and HDF and show how easy it is to create such beautiful apps with hardly any code.

## Step 1: Download and install the HDP Sandbox

Download and install the HDP 2.3 Sandbox on your laptop by following these instructions:

<http://hortonworks.com/products/hortonworks-sandbox/#install>

The Sandbox requires 8GB or RAM and 50GB of hard disk space. If your laptop doesn’t have sufficient RAM, you can launch it on Microsoft Azure cloud for free. Azure gives a free 30-day trial. Sign up for the free Azure trial and follow these instructions to install the Sandbox:

<http://hortonworks.com/blog/hortonworks-sandbox-with-hdp-2-3-is-now-available-on-microsoft-azure-gallery/> 

You’ll be asked to specify the VM size while creating the Sandbox on Azure. Please make sure to use A4 or higher VM sizes as the Sandbox is quite resource-intensive. Note down the VM username and password as you'll need it to log in to the Sandbox later.



## Step 2: Log in as root user 

You'll need root priveleges to finish this lab. There are different ways to log into the Sandbox as the root user, depending on whether it's running locally on your laptop or if it's running on Azure.

### Local Sandbox
Start the sandbox, and note its IP address shown on the VBox/VMware screen. Add the IP address into your laptop's "hosts" file so you don't need to type the Sandbox's IP address and can reach it much more conveniently at "sandbox.hortonworks.com". You will need to add a line similar to this in your hosts file:

```
192.168.191.241 sandbox.hortonworks.com sandbox    
```

Note: The IP address will likely be different for you, use that. 

The hosts file is located at `/etc/hosts` for Mac and Linux computers. For Windows it's normally at `\WINDOWS\system32\drivers\etc`. If it's not there, create one and add the entry as shown above.

The username for SSH is "root", and the password is "hadoop". You'll be the root user once logged in. Mac and Linux users can log in by:


```
$ ssh root@sandbox.hortonworks.com

```

Windows users can use the Putty client to SSH using "root" and "hadoop" as the username and password respectively. The hostname is "sandbox.hortonworks.com"

### Azure Sandbox
Note down the public IP address of the Sandbox VM from the Azure Portal UI. Add the IP address into your laptop's "hosts" file so you don't need to type the Sandbox's IP address and can reach it much more conveniently at "sandbox.hortonworks.com". You will need to add a line similar to this in your hosts file:

```
10.144.39.48 sandbox.hortonworks.com sandbox    
```

Note: The IP address will likely be different for you, use that. 

The hosts file is located at `/etc/hosts` for Mac and Linux computers. For Windows it's normally at `\WINDOWS\system32\drivers\etc`. If it's not there, create one and add the entry as shown above. 

Now we can log in to the sandbox.

Using the username which you entered while deploying the Sandbox on Azure, Mac and Linux users can log in by:

```
$ ssh <your_vm_username>@sandbox.hortonworks.com

```
The password is the VM password you specified while deploying the VM.

Windows users can use the Putty client to SSH and log in to "sandbox.hortonworks.com". Use the username and password you specified while creating the VM for SSH credentials. Once logged in, you need to assume the root user identity. Do this:

```
sudo su
```
Again, enter the same password you used to SSH in. Now you should be the root user.




## Step 2: Install NiFi and Solr on HDP Sandbox

Do the following on your Sandbox's shell prompt (make sure you're the root user!):

```
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
```

Note: From now on, type all code fragments like the one shown above on your Sandbox's shell prompt.

The command above checks for the HDP version number and stores it as a temporary environment variable. Next, download the Ambari NiFi service:

```
git clone https://github.com/abajwa-hw/ambari-nifi-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/NIFI
```   

Restart Ambari:

```
service ambari restart

```

Log in to Ambari UI, using "admin" as username and "admin" as password by opening the following URL in your Laptop's browser:

<http://sandbox.hortonworks.com:8080>

Click on "Add Service" from the "Actions" dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check NiFi server -> Next -> Next -> Next -> Deploy. By default NiFi port is set to 9090 and JVM memory size is 512MB. On successful deployment, you can log in to NiFi UI by going to:

<http://sandbox.hortonworks.com:9090/nifi>

Next, let's install Solr and configure the Banana dashboard.

Set JAVA_HOME:
```
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64
```

We'll use Banana as the dashboard and we've created its definition for you. Download it:

```
wget https://raw.githubusercontent.com/vzlatkin/Stocks2HBaseAndSolr/master/Solr%20Dashboard.json -O /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/banana/app/dashboards/default.json
```

Start Solr:

```
/opt/lucidworks-hdpsearch/solr/bin/solr start -c -z localhost:2181 
```

Create Solr collection:

```
/opt/lucidworks-hdpsearch/solr/bin/solr create -c stocks -d data_driven_schema_configs -s 1 -rf 1
```

We're now ready to use NiFi and create the flow.

## Step 3: Fetch stock prices from Google Finance 

Go to NiFi UI at <http://sandbox.hortonworks.com:9090/nifi>

Create a new process group by dragging the Process Group icon to the UI:

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/1-process-group.png)

Give it some name, eg: Stocks-Dashboard

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/2-create-process-group.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/3-add-get-http.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/4-configure-get-http.png)


## Step 4: Replace text

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/6-configure-replace-text.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/7-connect-gethttp-replace-text.png)

## Step 5: Split JSON

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/8-configure-split-json.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/9-connect-replacetext-splitjson.png)

## Step 6: Evaluate JSON 

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/10-configure-evaluate-json-path.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/11-connect-splitjson-evaluatejsonpath.png)

## Step 7: Update Attributes

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/12-configure-update-attribute.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/13-connect-evaluatejsonpath-updateattribute.png)

## Step 8: Prepare to Push to HBase and Solr

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/14-configure-attribute-json.png)

## Step 9: Push to HBase

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/Archive/15-puthbase-crreate-hbase-client-service.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/Archive/16-puthbase-addhbase-client-service.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/Archive/17-puthbase-clientservice-added.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/Archive/18-puthbase-enable-clientservice.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/Archive/19-puthbase-configure-client-service.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/20-puthbase-self-connection.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/21-puthbase-self-connected-result.png)



## Step 10: Push to Solr

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/22-configure-sendto-solr.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/23-configure-sendto-solr.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/24-configure-sendto-solr.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/25-connect-attributes-to-json-send-to-solr.png)

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/26-configure-logger.png)

## Step 11: Start the Flow and visualize in Banana

<http://sandbox.hortonworks.com:8983/solr/banana/index.html#/dashboard>

![](https://raw.githubusercontent.com/DhruvKumar/stocks-dashboard-lab/master/images/27-banana-dashboard.png)

Email: dkumar[*at*]hortonworks[*dot*]com




####  Ordered Lists

Ordered lists are created using "1." + Space:

1. Ordered list item
2. Ordered list item
3. Ordered list item

#### Unordered Lists

Unordered list are created using "*" + Space:

* Unordered list item
* Unordered list item
* Unordered list item 

Or using "-" + Space:

- Unordered list item
- Unordered list item
- Unordered list item

#### Hard Linebreak

End a line with two or more spaces will create a hard linebreak, called `<br />` in HTML. ( Control + Return )  
Above line ended with 2 spaces.

#### Horizontal Rules

Three or more asterisks or dashes:

***

---

- - - -




#### Tables

A simple table looks like this:

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell

If you wish, you can add a leading and tailing pipe to each line of the table:

| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |

Specify alignment for each column by adding colons to separator lines:

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right


### Shortcuts

#### View

* Toggle live preview: Shift + Cmd + I
* Toggle Words Counter: Shift + Cmd + W
* Toggle Transparent: Shift + Cmd + T
* Toggle Floating: Shift + Cmd + F
* Left/Right = 1/1: Cmd + 0
* Left/Right = 3/1: Cmd + +
* Left/Right = 1/3: Cmd + -
* Toggle Writing orientation: Cmd + L
* Toggle fullscreen: Control + Cmd + F

#### Actions

* Copy HTML: Option + Cmd + C
* Strong: Select text, Cmd + B
* Emphasize: Select text, Cmd + I
* Inline Code: Select text, Cmd + K
* Strikethrough: Select text, Cmd + U
* Link: Select text, Control + Shift + L
* Image: Select text, Control + Shift + I
* Select Word: Control + Option + W
* Select Line: Shift + Cmd + L
* Select All: Cmd + A
* Deselect All: Cmd + D
* Convert to Uppercase: Select text, Control + U
* Convert to Lowercase: Select text, Control + Shift + U
* Convert to Titlecase: Select text, Control + Option + U
* Convert to List: Select lines, Control + L
* Convert to Blockquote: Select lines, Control + Q
* Convert to H1: Cmd + 1
* Convert to H2: Cmd + 2
* Convert to H3: Cmd + 3
* Convert to H4: Cmd + 4
* Convert to H5: Cmd + 5
* Convert to H6: Cmd + 6
* Convert Spaces to Tabs: Control + [
* Convert Tabs to Spaces: Control + ]
* Insert Current Date: Control + Shift + 1
* Insert Current Time: Control + Shift + 2
* Insert entity <: Control + Shift + ,
* Insert entity >: Control + Shift + .
* Insert entity &: Control + Shift + 7
* Insert entity Space: Control + Shift + Space
* Insert Scriptogr.am Header: Control + Shift + G
* Shift Line Left: Select lines, Cmd + [
* Shift Line Right: Select lines, Cmd + ]
* New Line: Cmd + Return
* Comment: Cmd + /
* Hard Linebreak: Control + Return

#### Edit

* Auto complete current word: Esc
* Find: Cmd + F
* Close find bar: Esc

#### Post

* Post on Scriptogr.am: Control + Shift + S
* Post on Tumblr: Control + Shift + T

#### Export

* Export HTML: Option + Cmd + E
* Export PDF:  Option + Cmd + P


### And more?

Don't forget to check Preferences, lots of useful options are there.

Follow [@Mou](https://twitter.com/mou) on Twitter for the latest news.

For feedback, use the menu `Help` - `Send Feedback`