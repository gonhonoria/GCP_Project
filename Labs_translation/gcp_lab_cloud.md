# LAB: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL 


## Objectives

In this lab, you learn how to perform the following tasks:

    - Create a Cloud Storage bucket and place an image into it.

    - Create a Cloud SQL instance and configure it.

    - Connect to the Cloud SQL instance from a web server.

    - Use the image in the Cloud Storage bucket on a web page.

## Task 1: Sign in to the Google Cloud Platform (GCP) Console

## Task 2: Deploy a web server VM instance

- Create VM with startup script
	*gcloud compute instances create bloghost --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default  --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud*

	*gcloud compute  firewall-rules create allow-http --direction=INGRESS --action=ALLOW --rules=tcp:80  --target-tags=http-server*

## Task 3: Create a Cloud Storage bucket using the gsutil command line

1- Start Cloud Shell


2- For convenience, enter your chosen location into an environment variable called LOCATION. Enter one of these commands:

	*export LOCATION=US*


3- In Cloud Shell, the DEVSHELL_PROJECT_ID environment variable contains your project ID. Enter this command to make a bucket named after your project ID:

	*gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID*

4- Retrieve a banner image from a publicly accessible Cloud Storage location:

	*gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png*

5- Copy the banner image to your newly created Cloud Storage bucket:

	*gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png*

6- Modify the Access Control List of the object you just created so that it is readable by everyone:

	*gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png*

## Task 4: Create the Cloud SQL instance



- Create Cloud SQL instance

	*gcloud sql instances create blog-db --database-version=MYSQL_5_7 --region=us-central1 --root-password=password1234*

- Create a user

	*gcloud sql users create USERNAME --instance=blogdbuser  --password=password1234*

- Authorize(add VM Network for db access)

 	*export bloghostIP=$(gcloud compute instances describe bloghost-- format='get(networkInterfaces[0].accessConfigs[0].natIP))*

	*gcloud sql instances patch blog-db --authorized-networks=$bloghostIP/32*


## Task 5: Configure an application in a Compute Engine instance to use Cloud SQL

- SSH the VM instance bloghost

        gcloud compute ssh bloghost --zone=us-central1-a

- In your ssh session on bloghost, change your working directory to the document root of the web server:

	*cd /var/www/html*

- Use the nano text editor to edit a file called index.php:

	*sudo nano index.php*

- Paste the content below into the file:

	<html>
	<head><title>Welcome to my excellent blog</title></head>
	<body>
	<h1>Welcome to my excellent blog</h1>
	<?php
	 $dbserver = "CLOUDSQLIP";
	$dbuser = "blogdbuser";
	$dbpassword = "DBPASSWORD";
	// In a production blog, we would not store the MySQL
	// password in the document root. Instead, we would store it in a
	// configuration file elsewhere on the web server VM instance.

	$conn = new mysqli($dbserver, $dbuser, $dbpassword);

	if (mysqli_connect_error()) {
		echo ("Database connection failed: " . mysqli_connect_error());
	} else {
		echo ("Database connection succeeded.");
	}
	?>
	</body></html>

In a later step, you will insert your Cloud SQL instance's IP address and your database password into this file. For now, leave the file unmodified.

- Press Ctrl+O, and then press Enter to save your edited file.

- Press Ctrl+X to exit the nano text editor.

- Restart the web server:

	*sudo service apache2 restart*
	
- Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php. The URL will look like this:

	*curl $bloghostIP/index.php*

When you load the page, you will see that its content includes an error message beginning with the words:

Database connection failed: ...

This message occurs because you have not yet configured PHP's connection to your Cloud SQL instance.

- Return to your ssh session on bloghost. Use the nano text editor to edit index.php again.

	*sudo nano index.php*

- In the nano text editor, replace CLOUDSQLIP with the Cloud SQL instance Public IP address that you noted above. Leave the quotation marks around the value in place.

- In the nano text editor, replace DBPASSWORD with the Cloud SQL database password that you defined above. Leave the quotation marks around the value in place.

First in the cmd line interface:
        
	*export blogdbIP=$(gcloud compute instances describe blog-db-- format='get(networkInterfaces[0].accessConfigs[0].natIP))*

Then in nano text editor:

	<html>
	<head><title>Welcome to my excellent blog</title></head>
	<body>
	<h1>Welcome to my excellent blog</h1>
	<?php
	 $dbserver = $blogdbIP;
	$dbuser = "blogdbuser";
	$dbpassword = "password1234";
	// In a production blog, we would not store the MySQL
	// password in the document root. Instead, we would store it in a
	// configuration file elsewhere on the web server VM instance.

	$conn = new mysqli($dbserver, $dbuser, $dbpassword);

	if (mysqli_connect_error()) {
		echo ("Database connection failed: " . mysqli_connect_error());
	} else {
		echo ("Database connection succeeded.");
	}
	?>
	</body></html>

- Press Ctrl+O, and then press Enter to save your edited file.

- Press Ctrl+X to exit the nano text editor.

- Restart the web server:

	*sudo service apache2 restart*

- Return to the web browser tab in which you opened your bloghost VM instance's external IP address. 

	*curl $bloghostIP/index.php*

When you load the page, the following message appears:
Database connection succeeded.

In an actual blog, the database connection status would not be visible to blog visitors. Instead, the database connection would be managed solely by the administrator.


## Task 6: Configure an application in a Compute Engine instance to use a Cloud Storage object

- Check the bucket at the start of the project

	*gsutil ls -r gs://$PROJECT/my-excellent-blog.png*

- The object public link is the following: https://storage.googleapis.com/$DEVSHELL_PROJECT_ID/my-excellent-blog.png

- Return to your ssh session on your bloghost VM instance.

- Enter this command to set your working directory to the document root of the web server:

	*cd /var/www/html*

- Use the nano text editor to edit index.php:

	*sudo nano index.php*

- Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line.

- Paste this HTML markup immediately before the URL:

<img src='

    Place a closing single quotation mark and a closing angle bracket at the end of the URL:

'>

- The resulting line is:

	<img src='https://storage.googleapis.com/$DEVSHELL_PROJECT_ID/my-excellent-blog.png'>



- The effect of these steps is to place the line containing <img src='...'> immediately before the line containing <h1>...</h1>

	<html>
	<head><title>Welcome to my excellent blog</title></head>
	<body>
        <img src='https://storage.googleapis.com/$DEVSHELL_PROJECT_ID/my-excellent-blog.png'>
	<h1>Welcome to my excellent blog</h1>
	<?php
	 $dbserver = $blogdbIP;
	$dbuser = "blogdbuser";
	$dbpassword = "password1234";
	// In a production blog, we would not store the MySQL
	// password in the document root. Instead, we would store it in a
	// configuration file elsewhere on the web server VM instance.

	$conn = new mysqli($dbserver, $dbuser, $dbpassword);

	if (mysqli_connect_error()) {
		echo ("Database connection failed: " . mysqli_connect_error());
	} else {
		echo ("Database connection succeeded.");
	}
	?>
	</body></html>


- Press Ctrl+O, and then press Enter to save your edited file.

- Press Ctrl+X to exit the nano text editor.

- Restart the web server:

	*sudo service apache2 restart*

- Return to the web browser tab in which you opened your bloghost VM instance's external IP address. 
When you load the page, the web page content now includes a banner image.







