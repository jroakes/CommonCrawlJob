Getting Started
====================
It is recommended to utilize the latest version of CommonCrawlJob (directly from Github) rather than from the Pip repository.  

If you've already installed from Pip repository, start by entering:

.. code-block:: sh

   $ pip uninstall CommonCrawlJob

Then, install CommonCrawlJob:

.. code-block:: sh

   $ pip install git+https:////github.com/jroakes/CommonCrawlJob

Getting Familiar with the Dataset
---------------------------------
Take a glance at the dataset found in ``data/latest.txt``.  You'll find a big list of URL's like so:

::

	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00000-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00001-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00002-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00003-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00004-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00005-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00006-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00007-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00008-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00009-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00010-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00011-ip-10-236-182-209.ec2.internal.warc.gz
	common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00012-ip-10-236-182-209.ec2.internal.warc.gz
	...

These are all URL paths of, "chunks" of the total WARC data Common Crawl has collected from around the web (in this case, on its July 2016 crawl).  Each URL is referencing a location on Amazon S3 Public Datasets, where all of the Common Crawl data is hosted for free.  

If you'd like to **see** what the crawl data at a specific URL looks like, add:
``https://aws-publicdatasets.s3.amazonaws.com/`` in front of the URL.  

So, you could end up with something like the URL below, where you can download the data from this chunk and see for yourself: 
``https://aws-publicdatasets.s3.amazonaws.com/common-crawl/crawl-data/CC-MAIN-2016-07/segments/1454701145519.33/warc/CC-MAIN-20160205193905-00000-ip-10-236-182-209.ec2.internal.warc.gz``

While experimenting locally, it is recommended to limit the number of URL's in the ``data/latest.txt`` file to one or two URL's.  This can allow you to test locally on just 1-2 chunks of the total Common Crawl data, without overloading your own computer.

You can always visit the Common Crawl website to get the full list of WARC paths or regenerate this list after by running:

.. code-block:: sh

	# Generates some_file containing s3 buckets
	# some_file is the same file that you'll find in latest.txt
	python -m aws > some_file.txt

An Example Extractor
====================

Let's build a simple extractor for Google Analytics Trackers.

First let's create a file ``GoogleAnalytics.py``

.. code-block:: sh

   $ touch GoogleAnalytics.py

We will go from start to finish in creating a Common Crawl extractor that uses regular expression capture groups to extract
google analytics tracker id's.

Our ``GoogleAnalytics`` class has is overriding one method ``mapper_init`` which defines a compiled regular expressions
that will be matched over the HTML content.

All common crawl jobs will generally obey this pattern.


Running Locally
---------------

Run the Google Analytics extractor locally to test your script.

.. code-block:: sh

    $ python GoogleAnalytics.py -r local <(tail -n 1 data/latest.txt)

If you're having trouble running the code using the command above, try:

.. code-block:: sh

    $ [sudo] python GoogleAnalytics.py < data/latest.txt

In this command, python is running the code for ``GoogleAnalytics.py``, by downloading Common Crawl data from the list of URL's found on ``data/latest.txt``. You'll notice this outputs the results to your command line.  

In this case, removing the -r local tag will actually try to simulate a Hadoop streaming-ish environment, but can introduce some issues.  It is ok for our purposes though, at the moment.

You can then run the below command to get ``GoogleAnalytics.py`` to save the output to a file:

.. code-block:: sh

    $ [sudo] python GoogleAnalytics.py < data/latest.txt | tail >> output.txt

Sample command line output:

::

	Using configs in /etc/mrjob.conf
	Creating temp directory /tmp/GoogleAnalytics.root.20160726.225707.263660
	Running step 1 of 1...
	reading from STDIN
		Counters: 1
			commoncrawl
			processed_record=12261
		Counters: 1
			commoncrawl
			processed_record=12261
	Streaming final output from /tmp/GoogleAnalytics.root.20160726.225707.263660/output...
	Removing temp directory /tmp/GoogleAnalytics.root.20160726.225707.263660...

Sample output.txt file:

::
	
	"http://ccad.edu.websiteoutlook.com/"	"68038641"
	"http://ccc.edu/pages/file-not-found.aspx?url=/departments/events/menu/Pages/Success-Stories.aspx"	"20438531"
	"http://ccc.edu/pages/file-not-found.aspx?url=/departments/events/menu/Pages/Success-Stories.aspx"	"8735345"
	"http://cccreationsusa.com/?ATCLID=206493962&SPID=93247&DB_OEM_ID=27300"	"7776403"
	"http://ccdl.libraries.claremont.edu/cdm/search/collection/cce/searchterm/\u201cThe/mode/all/order/descri"	"60555116"
	"http://ccdl.libraries.claremont.edu/cdm/search/collection/cce/searchterm/candidates/mode/all/order/descri"	"60555116"
	"http://ccdl.libraries.claremont.edu/cdm/search/collection/lea/searchterm/Celebrating"	"60555116"
	"http://ccdl.libraries.claremont.edu/cdm/search/collection/lea/searchterm/treats"	"60555116"
	"http://ccherb.com/category/vaginal-infections/"	"22470526"
	"http://cctvdirectbuy.com/index.php?main_page=product_info&cPath=77&products_id=1461"	"10097023"

Elastic Map Reduce - Configuration
==================================

You're now ready to run the ``GoogleAnalytics.py`` script on Amazon EMR!

Setup your requirements.txt file:
---------------------------------
Using the existing requirements.txt template in this repo, add the following line, and save the file:

::

	-e git://github.com/jroakes/CommonCrawlJob.git#egg=CommonCrawlJob

This line will install the latest version of CommonCrawl Job on EMR as it is preparing the instance and running its, "bootstrap actions".  

Visit the AWS website and access the S3 web panel.  Create a new bucket on Amazon S3 and call it something like:

::
	
	your-name-emr-scripts

Click, "upload" inside of the new bucket you have created and upload your requirements.txt file.  Make sure to set permissions, so that, the "everyone" group can open/view the file.

Configuring your own environment
--------------------------------
For best performance, you should launch the cluster in the same region
as your data. Currently data from ``aws-publicdatasets`` are stored in
``us-east-1``, which is where you want to point your EMR cluster.

NOTE: EMR is a service that costs money, so, always use this service with caution by monitoring your own expenses.  The authors of this tutorial/software are not responsible for any expenses/damages incurred on your AWS account.  Use at your own risk.

Common Crawl Region
-------------------
:S3: US Standard
:EMR: US East (N. Virginia)
:API: ``us-east-1``

Create an Amazon EC2 Key Pair and PEM File
------------------------------------------

Amazon EMR uses an Amazon Elastic Compute Cloud (Amazon EC2) key pair
to ensure that you alone have access to the instances that you launch.

The PEM file associated with this key pair is required to ssh directly to the master node of the cluster.

To create an Amazon EC2 key pair:
---------------------------------

.. image:: /static/img/EC2KeyPair.png
   :alt: EC2 Key Pair
   :align: center

1. Go to the Amazon EC2 console
2. In the Navigation pane, click Key Pairs
3. On the Key Pairs page, click Create Key Pair
4. In the Create Key Pair dialog box, enter a name for your key pair, such as, mykeypair
5. Click Create
6. Save the resulting PEM file in a safe location

Configuring ``mrjob.conf``
--------------------------

Make sure to download an EC2 Key Pair ``pem`` file for your map reduce
job and add it to the ``ec2_key_pair`` and ``ec2_key_pair_file``
variables.

Make sure that the ``PEM`` file has permissions set properly by running

.. code-block:: sh

    $ chown 600 $MY_PEM_FILE


Create a ``mrjob.conf`` file to set up your configuration parameters to match
that of AWS.

There is a default configuration template located at ``mrjob.conf.template`` that you can use.  

To run on EMR, use the following:

.. code-block:: sh

	python3 GoogleAnalytics.py -r emr --conf-path="mrjob.conf" --no-strict-protocols --output-dir='s3://s3-bucket/folder' data/run-min.txt


``--no-strict-protocols`` will skip pages with encoding errors via mrjob. 

Navigate to the bucket name you created when entering the command and you should see your results output inside of the bucket.  EMR has already split the output results into multiple files.

Here's a sample of some output inside of the ``part-00000.gz`` file:

::

	"http:\/\/1012lounge.com\/"	"3479925"
	"http:\/\/102theriver.iheart.com\/articles"	"45084235"
	"http:\/\/1065ctq.iheart.com\/articles\/national-news-104668\/new-electronic-license-plates-could-be-11383289\/"	"45084235"
	"http:\/\/12160.info\/group\/gunsandtactics\/forum\/topic\/show?id=2649739:Topic:1105218&xg_source=msg"	"2024191"
	"http:\/\/1350kman.com\/settlement-reached-in-salina-contamination-cleanup\/"	"47189603"
	"http:\/\/2670.antiquesnavigator.com\/index.php?main_page=product_info&cPath=124&products_id=4628"	"212281"
	"http:\/\/303magazine.com\/2012\/10\/undead-mans-party-casselmans-hosts-zombie-crawl-aftermath-featuring-celldweller\/"	"19049911"
	"http:\/\/511.ky.gov\/kylb\/roadreports\/event.jsf?sitKey=kycars4-10068&view=state&text=l&textOnly=false&current=true"	"16329038"
	"http:\/\/610wtvn.iheart.com\/articles\/entertainment-news-104658\/fun-to-perform-with-queen-at-11672188\/"	"45084235"
	"http:\/\/7warez.mihanblog.com\/post\/tag\/\u062f\u0627\u0646\u0644\u0648\u062f+KMPlayer+3"	"153829"
	"http:\/\/949thebull.iheart.com\/onair\/caffeinated-radio-jason-pullman-kristen-gates-30022\/celeb-perk-11513-11797938\/"	"45084235"
	"http:\/\/965tic.cbslocal.com\/2011\/05\/05\/cinco-de-mayo-top-10-tasty-mexican-inspired-cocktails-for-your-feista-3\/3\/"	"19997300"
	"http:\/\/987ampradio.cbslocal.com\/2011\/02\/13\/cee-lo-green-gwyneth\/"	"20574553"
	"http:\/\/987ampradio.cbslocal.com\/tag\/going-out\/"	"20574553"
	"http:\/\/99mpg.com\/workshops\/understandingdiagn\/firststart\/"	"43148446"
	...


You have now successfully ran your first EMR job over part of the Common Crawl Archive!

Next Steps
----------
When ready, run the above command over every URL inside of the original ``data/latest.txt`` file.   This will run ``GoogleAnalytics.py`` over the ENTIRE Common Crawl July 2016 archive.  

Proceed with caution as this is a bigger EMR job and could be costly ... continue at your own risk ;)  Best of luck!!
