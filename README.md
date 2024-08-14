# Conversion Status Checker

This is not an officially supported Google product. 

Conversion Status Checker leverages a custom server tag that builds a data pipeline to BigQuery instance to help you troubleshoot conversions and better understand share of consented and unconsented conversions.

**What Conversion Status Checker can do:**
* Visualize the share of consented and unconsented data that is being sent through sGTM. Data can be visualised in Looker Studio or brought into the BI system of choice for further analysis and insights
* Unconsented data is sent only with Advanced Consent Mode implementation. In order for this solution to work, you need to have Advanced Consent Mode implemented on website. If you have Basic Consent Mode, implementing this solution you will get only consented data collected in BigQuery

**What Conversion Status Checker can NOT do:**
* It does not provide insights into observed vs. modelled data
* It does not decode gcd parameter. Gcd parameter is exported to into BigQuery table, so you can use it to make your own analysis



## How to set up the solution

Follow these steps to set up the solution:
* Join this group to get access to JSON file (any potential updates will be published in the group)
* Download the [JSON file](https://github.com/googlestaging/conversion-status-checker/blob/main/conversion-status-checker.json)
* Follow implementation guide (explained below)



# Implementation guide

Prerequisites:
* Advanced Consent Mode implemented on website
* sGTM container up and running
* Access to Google Cloud Platform (BigQuery)
* Looker Studio or some other visualization tool that can be connected to BigQuery

NOTE: It is worth mentioning that deploying this solution could occur additional sGTM and GCP costs.


The process for setting up the solution is following:
1. Create new dataset in BigQuery
2. Provide permissions for Service account
3. Enable BigQuery API
4. Import tag in sGTM container
5. Configure tags in sGTM
6. Copy Looker Studio dashboard


## 1. Create new dataset in BigQuery
Go to BigQuery in your Google Cloud Platform project where you want to store the data and create new dataset:
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/bigquery-dataset.png"></p>

In newly created dataset, create new table:
* Provide table name
* Enable toggle "Edit as text" and copy the table schema from [this file](https://github.com/googlestaging/conversion-status-checker/blob/main/bigquery-table-schema.txt).
* Select "Partition by ingestion time"
* Select "By day" as Partitioning type
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/bigquery-table.png"></p>

Save the table. This is how your table should look like:
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/bigquery-table-review.png"></p>


## 2. Provide permissions for Service account
In order to be able to write data in BigQuery, you need to provide permissions for your Service Account. Go to the GCP Project where you deployed your sGTM container. Navigate to **IAM & Admin** -> **Service Accounts** and copy your Service Account email.

Next, navigate to the GCP Project where you want to store data in BigQuery. Even if you are using the same GCP project for sGTM container and for storing data in BigQuery, you still need to do this step. Navigate to **IAM & Admin** -> **IAM**. Click on **Grant Access** button. 

Paste your Service Account in **New principals** field and assign **BigQuery Data Editor** role. Click on Save button.
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/service-account.png"></p>


## 3. Enable BigQuery API
In GCP search bar, type "BigQuery API" and open the page. If you haven't already, click on button and enable the API. 
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/bigquery-api.png"></p>


## 4. Import tag in sGTM container
Download the [JSON file](https://github.com/googlestaging/conversion-status-checker/blob/main/conversion-status-checker.json).

Go to [tagmanager.google.com](https://tagmanager.google.com/) and open you server-side container where you want to implement the solution. 
Go to **Admin** -> **Import Container** and import the JSON file you just downloaded.
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/gtm-import-container.png"></p>


## 5. Configure tags in sGTM
After you successfully imported the JSON file, click on the left side navigation on **Tags** and open **Conversion Status Checker** tag. 
In the tag, provide your:
* GCP Project ID
* Dataset ID
* Table ID
* Set trigger for the tag (you can fire the tag on every single event or only on specific events, the choice if yours and depends on your use case)


In order to be able to get tag information in BigQuery, you need to enable Tag Metadata for every single tag that you have in sGTM container. Looker Studio dashboard (explained in next step) that is used to visualize data from BQ table has some predefined filters. To make sure those filters work properly, follow the steps how to provide Metadata for every Google Ads Conversion, Floodlight and Google Analytics tags in sGTM container:

**Open every Google Ads Conversion tag** in sGTM container, enable Tag Metadata and provide following information:
* In the field "Key for tag name" enter **name**
* Add additional Metadata key **type**, and as value enter **Google Ads Conversion**
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/g-ads-metadata.png"></p>

Next, **open every Floodlight tag** in sGTM container, enable Tag Metadata and provide following information:
* In the field "Key for tag name" enter **name**
* Add additional key **type**, and as value enter **Floodlight**
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/floodlight-metadata.png"></p>

Next, **open every Google Analytics tag** in sGTM container, enable Tag Metadata and provide following information:
* In the field "Key for tag name" enter **name**
* Add additional key **type**, and as value enter **Google Analytics**
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/ga-metadata.png"></p>


It is only important to provide exact **type** value as discribed above for Floodlight, Google Ads Conversion and GA4 tags, because for these three tag types Looker Studio dashboard has some predefined filters that align on those values.

In case if you have other tags in sGTM, you need to provide type Metadata as well. But, for these tags, feel free to provide **type** value as you prefer. For example, if you have Meta tag, feel free to add type metadata and define value as you prefer (e.g. Facebook CAPI, Facebook, Facebook event etc.).

In case if you have Google Ads Remarketing tags, feel free to put any value you prefer (G Ads Remarketing, Ads Remarketing etc.). Example:
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/other-tags-metadata.png"></p>


If you have certain tags that you don't want to collect information in BigQuery (for example, you don't want to collect information for Conversion Linker tag). In this case, just provide key **exclude** and value as **true** for those tags you don't want to collect info in BigQuery:
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/exclude-metadata.png"></p>


Once you are done, publish the container.


## 6. Copy Looker Studio dashboard
Go to [Looker Studio](https://lookerstudio.google.com/c/navigation/reporting), click on **Create** button in upper left corner and select **Data source**.
Select BigQuery as Connector and find your BigQuery table:
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/looker-studio-data-source.png"></p>

**NOTE:** After you successfully created a Looker Studio Data Source, wait 2-3 minutes before copying the dashboard. Sometimes it need some time to sync before your newly created data source is visible in the dropdown menu when you copy the dashboard.

Open [Looker Studio dashboard](https://lookerstudio.google.com/s/vHO_sFsW7K8). In upper right corner, click on three dots and then on **Make a copy**. In new data source dropdown, find and select data source you just created and click on **Copy report**.
<p align="center"><img src="https://github.com/googlestaging/conversion-status-checker/blob/main/images/dashboard-copy.png"></p>




































