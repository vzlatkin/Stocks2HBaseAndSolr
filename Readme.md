<h1>Visualize near-real-time stock price changes using Solr and Banana UI</h1>
<p>
	The goal of this tutorial is to create a moving chart that shows the changes in price of a few stock symbols, similar to Google Finance or Yahoo Finance.
</p>
<h2>Summary of steps</h2>
<ol>
	<li>Download and install the HDP Sandbox </li>
	<li>Download and install the latest NiFi release </li>
	<li>Create a Solr dashboard to visualize the results</li>
	<li>Create a new NiFi flow to pull from Google Finance API, transform, and store in HBase and Solr</li>
</ol>
<h2>Step-by-step </h2>
<h3>1. Download and install the HDP Sandbox</h3>
<p>
	Download the latest (2.3 as of this writing) HDP Sandbox 
	<a href="http://hortonworks.com/products/hortonworks-sandbox/#install.">here</a>. Import it into VMware or VirtualBox, start the instance, and update the DNS entry on your host machine to point to the new instance’s IP.
</p>
<p>
	On Mac, edit 
	<em>/etc/hosts</em>, on Windows, edit <em>%systemroot%\system32\drivers\etc\</em> as administrator and add a line similar to the below:
</p>
<pre>
192.168.56.102  sandbox sandbox.hortonworks.com
</pre>
<h3></h3>
<h3>2. Download and install the latest NiFi release</h3>
<p>
	Follow the directions 
	<a href="https://nifi.apache.org/docs.html">here</a>.  These were the steps that I executed for 0.4.1
</p>
<pre>
cd /tmp
wget http://apache.cs.utah.edu/nifi/0.4.1/nifi-0.4.1-bin.zip
cd /opt/
unzip  /tmp/nifi-0.4.1-bin.zip
useradd nifi
chown -R nifi:nifi /opt/nifi-0.4.1/
perl -pe 's/run.as=.*/run.as=nifi/' -i /opt/nifi-0.4.1/conf/bootstrap.conf
perl -pe 's/nifi.web.http.port=8080/nifi.web.http.port=9090/' -i /opt/nifi-0.4.1/conf/nifi.properties
/opt/nifi-0.4.1/bin/nifi.sh start
</pre>
<h3>3. Create a Solr dashboard to visualize the results</h3>
<p>
	Download a new Solr dashboard, start the service, and create a new collection to store stock price changes:
</p>
<pre>
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64
wget https://raw.githubusercontent.com/vzlatkin/Stocks2HBaseAndSolr/master/Solr%20Dashboard.json -O /opt/lucidworks-hdpsearch/solr/server/solr-webapp/webapp/banana/app/dashboards/default.json
/opt/lucidworks-hdpsearch/solr/bin/solr start -c -z localhost:2181 
/opt/lucidworks-hdpsearch/solr/bin/solr create -c stocks -d data_driven_schema_configs -s 1 -rf 1
</pre>
<h3>4. Create a new NiFi flow to pull from Google Finance API, transform, and store in HBase and Solr</h3>
<p>
	Solr is used for indexing the data, Banana UI is used for visualization, and HBase is used for future-proofing.  HBase can be used to further analyze the data from Storm/Spark or to create a custom UI.  The get the data into these tools, follow the steps below:
</p>
<ul>
	<li>Start HBase via <a href="http://sandbox.hortonworks.com:8080/#/main/services/HBASE/summary">Ambari</a></li>
	<li>Create a new table:
	<pre>
hbase shell
hbase(main):001:0&gt; create 'stocks', 'cf'
	</pre>
	</li>
	<li>
	Then download 
	<a href="https://raw.githubusercontent.com/vzlatkin/Stocks2HBaseAndSolr/master/Send_stock_prices_to_HBase_and_Solr.xml">this NiFi template</a> to your host machine.
	</li>
	<li>To import the template, open the <a href="http://sandbox.hortonworks.com:9090/nifi/">NiFi UI</a></li>
	<li>Open Templates manager:
	<p>
		<img src="/storage/attachments/1144-import-into-nifi-step-1.png" style="background-color: initial;">
	</p>
	</li>
	<li>
	<p>
		Find the template on your local machine and import it:
	</p>
	<p>
		<img src="/storage/attachments/1128-import-into-nifi-step-2.png" style="width: 489px;">
	</p>
	</li>
	<li>
	<p>
		Drag and drop to instantiate a new template:
	</p>
	<p>
		<img src="/storage/attachments/1130-import-into-nifi-step-3.png" style="background-color: initial;">
	</p>
	</li>
	<li>
	<p>
		Double click the new process group:
	</p>
	<p>
		<img src="/storage/attachments/1141-import-into-nifi-step-4.png" style="background-color: initial;">
	</p>
	</li>
	<li>
	<p>
		You'll need to enable the HBase shared controller.  To do so, click the right mouse button over the "Send to HBase" process, then click "Configure", then "Properties" and the "Go to" arrow to access the controller. Finally, click the "Enable" button.
	</p>
	<p>
		<img src="/storage/attachments/1142-import-into-nifi-step-5.png" style="width: 619px;">
	</p>
	</li>
	<li>
	<p>
		Now start all of the processes.  Hold down the Shift-key, and select all of the processes on the screen.  Then click the start button:
	</p>
	<p>
		<img src="/storage/attachments/1143-import-into-nifi-step-6.png">
	</p>
	</li>
</ul>
<p>
	You should see a flow that looks like the below screenshot
</p>
<p>
	<img src="/storage/attachments/1145-nifi-screenshot.png" style="background-color: initial;">
</p>
<p>
	The reason for so many processes is that the response from Google Finance API needs to be transformed.  First, we remove the comment characters '//' from the response.  Second, we split the array into individual JSON objects.  Third, we extract the relevant attributes.  Fourth, the timestamp has the format of UTC, but it is actually in EST timezone, therefore, we fix that.  Finally, we send the information to HBase, Solr, and the NiFi bulletin board for logging.
</p>
<h2>Conclusion</h2>
<p>
	Now open the 
	<a href="http://sandbox.hortonworks.com:8983/solr/banana/index.html#/dashboard">Banana UI</a>.  If you are doing this when the US stock markets are open (9:30am to 4pm Eastern Time), then you should see a dashboard similar to the below.
</p>
<p>
	<img src="/storage/attachments/1146-banana-screenshot.png">
</p>
