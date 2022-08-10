<p style="text-align: justify;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;"><span style="background-color: white;">This user guide is </span>all about configuration and
deployment of telemetry streaming service on F5 BIG-IP device and scraps those
metrics by Prometheus which will be finally visualized by the Grafana. One can
select the relevant metrics scraped by the Prometheus and visualize them on the
Grafana which will be demonstrated later in the guide.</span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">This guide is heavily based on the work performed by
Michael O'Leary and one can view on <a href="https://github.com/mikeoleary/prometheus-ts-cis-kic-guide" target="_blank">here</a>. The purpose of this guide is to
document a little more elaborated guide for both learning and deployment
aspects and also address the possible issues that could be faced during the process
of deployment.<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<p class="MsoNormal" style="text-align: justify;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Telemetry streaming (TS)</b> is an iControl LX extension
delivered as a TMOS-independent RPM file with the ability to declaratively
aggregate, normalize and forward statistics and events from the BIG-IP to a
consumer application by posting a single TS JSON declaration to TS’s
declarative REST API endpoint. Additional information about Telemetry streaming
can be found on<o:p></o:p></span></p>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto;"><tbody><tr><td style="text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjWIMXWAQKp1JW2KzJlVm6Etzen-I6KA8-suz1ou7S_mhuDSn7GOdNjRxYSnk5eXTiUgrtxaawWqon_Y3AZmqEZ_mGeBqxWtdXylKTgj4vR0oBMg6BzNH5U17LCyeSe10CBM7pzm38wK5fuLuuwKtMFldNen6gMkmM9ngCnv0aDe18K-RKJi7TdVRMG/s3344/telemetry%20streaming.png" style="margin-left: auto; margin-right: auto;"><img border="0" data-original-height="1252" data-original-width="3344" height="240" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjWIMXWAQKp1JW2KzJlVm6Etzen-I6KA8-suz1ou7S_mhuDSn7GOdNjRxYSnk5eXTiUgrtxaawWqon_Y3AZmqEZ_mGeBqxWtdXylKTgj4vR0oBMg6BzNH5U17LCyeSe10CBM7pzm38wK5fuLuuwKtMFldNen6gMkmM9ngCnv0aDe18K-RKJi7TdVRMG/w640-h240/telemetry%20streaming.png" width="640" /></a></td></tr><tr><td class="tr-caption" style="text-align: center;">Diagram of BIG-IP or API service Gateway</td></tr></tbody></table><p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Reference to the above diagram can be found <a href="https://clouddocs.f5.com/training/community/automation/html/class05/postmanDeployment/module4/lab1.html" target="_blank">here</a>.</span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /></span></p><p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Prometheus is an open-source monitoring solution
that stores time series data like metrics whereas Grafana allows visualizing
the data stored in Prometheus and also supports a wide range of other sources.</span></p>

<h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /></span></h2><h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Architecture diagram used on the deployment </span></h2>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhAwDk4TOnGJqkOrnVJCW0QPMfvJTJdri1zBCREKfjy67JO3RvskJHBZp246eBUvUv_yj6lqRcwKxIHKPMfLzs2BkWeIoIl3Miezz8Q5qjUYklmOyY5pAkI0klHWb6s-an57ZVnp8utf6hGz3zYeWFthZuytCHpnBYeA2qnOnTi_mviINHM0DfYXwwL/s631/architecture%20diagram%20of%20telemetry%20streaming.jpg" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="354" data-original-width="631" height="360" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhAwDk4TOnGJqkOrnVJCW0QPMfvJTJdri1zBCREKfjy67JO3RvskJHBZp246eBUvUv_yj6lqRcwKxIHKPMfLzs2BkWeIoIl3Miezz8Q5qjUYklmOyY5pAkI0klHWb6s-an57ZVnp8utf6hGz3zYeWFthZuytCHpnBYeA2qnOnTi_mviINHM0DfYXwwL/w640-h360/architecture%20diagram%20of%20telemetry%20streaming.jpg" width="640" /></a></div><br /><p class="MsoNormal"><br /></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">A short briefing about the architecture diagram in
case of this user-deployment case scenario, the F5 BIG-IP system is on
standalone mode with a management IP of 172.20.100.173, and both Prometheus and
Grafana services are running on the same host with an IP address of
192.168.180.191 where the service port for Prometheus is on default – 9090 and
the service port for Grafana is 5000.<o:p></o:p></span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhIYqZ0z5VsGEcKtWSo3VGl1I9genaUkerIggwZY_2-XvJngnCmUoyOdQDKyynbexuTF79842MoMWe65g86tLcTflRKrYkKeTsXuv6l3vs_9JTrRZ0ESVMVtJxuQhFqwVNGbw6qZGy_WoRxIB5HfaKYgwBBRvzvuqoG7Jqf9Oq-xbBwXFvb5ZAwBdcY/s618/architecture%20diagram%20of%20telemetry%20streaming%20(user-guide).jpg" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="371" data-original-width="618" height="384" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhIYqZ0z5VsGEcKtWSo3VGl1I9genaUkerIggwZY_2-XvJngnCmUoyOdQDKyynbexuTF79842MoMWe65g86tLcTflRKrYkKeTsXuv6l3vs_9JTrRZ0ESVMVtJxuQhFqwVNGbw6qZGy_WoRxIB5HfaKYgwBBRvzvuqoG7Jqf9Oq-xbBwXFvb5ZAwBdcY/w640-h384/architecture%20diagram%20of%20telemetry%20streaming%20(user-guide).jpg" width="640" /></a></div><div class="separator" style="clear: both; text-align: center;"><br /></div>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">The whole deployment guide is broadly divided into the following sections and one can jump to the required step if they have achieved
the previous configuration successfully:<o:p></o:p></span></p>

<h2><ul style="text-align: left;"><li><span style="font-size: 16px;">Section I: Download and install Telemetry Streaming</span></li><li><span style="font-size: 16px;">Section II: Telemetry Streaming Declaration on the F5 BIG-IP device</span></li><li><span style="font-size: 16px;">Section III: Configuration of Prometheus</span></li><li><span style="font-size: 16px;">Section IV: Configuration on Grafana using Prometheus as a data source</span></li></ul></h2><h2><span style="font-size: 16px;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="line-height: 107%; mso-bidi-font-size: 10.0pt;"></span></span></h2><div><span style="font-size: 16px;"><br /></span></div><div><span style="font-size: 16px;"><br /></span></div><h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 12pt; line-height: 107%; mso-bidi-font-size: 10.0pt;">Section I: Download and
install Telemetry Streaming</span></h2>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">We need to first download and install the telemetry
streaming package on the F5 BIG-IP device. Since the telemetry streaming
package is an RPM file that can be downloaded and can install through GUI
or curl command on the CLI of the F5 BIG-IP device.<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">In this user manual guide, we will download and then
upload the telemetry streaming package on the BIG-IP using the iControl/iApp LX
framework.<o:p></o:p></span></p>

<i><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">One can use the alternative way which can be found&nbsp;<a href="http://here." target="_blank">here.</a></span></i><br />

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">At first, we need to download the RPM file, one can
find the latest telemetry streaming RPM file on the F5 Telemetry site on GitHub and
download the latest RPM file. <o:p></o:p></span></p><p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">The GitHub page to download telemetry streaming can be found&nbsp;<a href="https://github.com/F5Networks/f5-telemetry-streaming/releases" target="_blank">here</a>.</span></p>

<p class="MsoNormal"><br /></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">After downloading the file, you need to access your
F5 BIG-IP GUI with your admin privilege account then follow the following
steps:<o:p></o:p></span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhxbzgz69kUIEVcKkoljN4Y7z9nPcax6ffbzjj0Y4X7GU234L122bjAYnzwIpvjIXjm0zYFhpQa-x6p8y4qsJpVlmNKPYIXZ1s2kXNGYSxTxJWXfoMjkKaYYDFeMVU5Gey_ZyLpTkB38fg_zhabO6tlAjsVIUpZ2w_n1O_oB9AijJbEsYw6kyx2dNCa/s1761/fresh%20installation%20of%20telemetry%20streaming%20package%20on%20f5.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="620" data-original-width="1761" height="226" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhxbzgz69kUIEVcKkoljN4Y7z9nPcax6ffbzjj0Y4X7GU234L122bjAYnzwIpvjIXjm0zYFhpQa-x6p8y4qsJpVlmNKPYIXZ1s2kXNGYSxTxJWXfoMjkKaYYDFeMVU5Gey_ZyLpTkB38fg_zhabO6tlAjsVIUpZ2w_n1O_oB9AijJbEsYw6kyx2dNCa/w640-h226/fresh%20installation%20of%20telemetry%20streaming%20package%20on%20f5.png" width="640" /></a></div><div class="separator" style="clear: both; text-align: center;"><br /></div><br /><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj71d3VNOsPOnl6SVJM5Es32aF-lL_3j8In-glxV-3JorZK5KGk0Puws_cjabgxV8KFUxXxegAa6LUsGytOTWq0IykYkMx_vbVQwP-30z5Tnj1Q1YxGzCqDrj3FCuLKlozaj52byAajZEzoboH5h4UzLBXJk_jKzhiPohfSR1dRCNZIO9g-MF2QtG_C/s1821/successfully%20installed%20telemetry%20streaming.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="520" data-original-width="1821" height="182" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj71d3VNOsPOnl6SVJM5Es32aF-lL_3j8In-glxV-3JorZK5KGk0Puws_cjabgxV8KFUxXxegAa6LUsGytOTWq0IykYkMx_vbVQwP-30z5Tnj1Q1YxGzCqDrj3FCuLKlozaj52byAajZEzoboH5h4UzLBXJk_jKzhiPohfSR1dRCNZIO9g-MF2QtG_C/w640-h182/successfully%20installed%20telemetry%20streaming.png" width="640" /></a></div><br /><p class="MsoNormal"><br /></p>

<h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 12pt; line-height: 107%; mso-bidi-font-size: 10.0pt;">Section II: Telemetry
Streaming Declaration on the F5 BIG-IP device</span></h2>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Once the download and installation of the F5 telemetry
streaming package have been completed, we need to send a Telemetry Streaming
declaration to configure a Telemetry Streaming pull consumer target. <o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Before we jump into this configuration, we need to
create a new user with an administrator role on the F5 BIG-IP device and you can
just continue with the default admin user on the further configuration.<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">We can create a new user in the following steps:</span></p><p class="MsoNormal"></p><ul style="text-align: left;"><li><span style="font-size: 12pt; text-indent: -0.25in;"><b>Go to System &gt; Users &gt; User List</b></span></li><li><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Click on Create button<o:p></o:p></b></span></li><li><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Input the new user’s name and password<o:p></o:p></b></span></li><li><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Select role as administrator then add</b></span></li><li><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Click on the Finished button</b></span></li></ul><p></p>

<p class="MsoListParagraphCxSpMiddle" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--></p>

<p class="MsoListParagraphCxSpMiddle" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--></p>



<p class="MsoListParagraphCxSpLast" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--></p>

<div class="separator" style="clear: both; text-align: center;"><br /></div><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiSGvA6i6aJl83Y06MiQ73ByV_oCReRuFoXjVcuDw1IB8cJNAaTygssRe7vDETJuvFuHdCiT5oEeh90Dgi86xks7bK1LqvPRbrt9C7vQxgbTJGWZ0encuBGg_ZGfriOlkMTcMmqoZBpyoAn6MNNoRchYWVBieSFYD5rJlktofkuwygqnOfEe7xtsnfj/s1476/user%20created.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="275" data-original-width="1476" height="120" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiSGvA6i6aJl83Y06MiQ73ByV_oCReRuFoXjVcuDw1IB8cJNAaTygssRe7vDETJuvFuHdCiT5oEeh90Dgi86xks7bK1LqvPRbrt9C7vQxgbTJGWZ0encuBGg_ZGfriOlkMTcMmqoZBpyoAn6MNNoRchYWVBieSFYD5rJlktofkuwygqnOfEe7xtsnfj/w640-h120/user%20created.png" width="640" /></a></div><p class="MsoNormal"><br /></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">As we’re using Prometheus on this user-guide manual
so, the Telemetry Streaming consumer target will be Prometheus which is hosted
on 192.168.180.191:5000<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">We can either use Postman or using curl command on the CLI of the F5 BIG-IP device to configure a Telemetry Streaming pull consumer
target.<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 14pt; line-height: 107%; mso-bidi-font-size: 11.0pt;">&nbsp;</span></p>

<h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 14pt; line-height: 107%; mso-bidi-font-size: 11.0pt;">Configuration using
Postman application</span></h2>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Just follow the following steps for the configuration
of the telemetry streaming consumer target using the Postman application.<o:p></o:p></span></p>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step I: Open the Postman and create a new tab</span></h3>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step II: Select the GET method and paste the
following link</span></h3>

<p class="MsoNormal"><span class="MsoHyperlink"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">https://&lt;big-ip-management-ip-address&gt;/mgmt/shared/telemetry/declare</span></span><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiYDDhfOmBQMZSNMgZIXx8taSUy0qCPQrm-2SuRtwiB6mJzSZk2Do2nhl6jg0f5D31jFVYKz_fhtsKhustR_Y5BJWob-R__ifT6-0mGU02TzHPBBoEIYwZBlsAGHwoXSUt0aOFtA8t7hpJqYl_kkRIFw5jrV9-hFjNHNv6bC4grOVUTWwnoMfW9QI97/s1278/new%20PostMan%20first.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="424" data-original-width="1278" height="212" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiYDDhfOmBQMZSNMgZIXx8taSUy0qCPQrm-2SuRtwiB6mJzSZk2Do2nhl6jg0f5D31jFVYKz_fhtsKhustR_Y5BJWob-R__ifT6-0mGU02TzHPBBoEIYwZBlsAGHwoXSUt0aOFtA8t7hpJqYl_kkRIFw5jrV9-hFjNHNv6bC4grOVUTWwnoMfW9QI97/w640-h212/new%20PostMan%20first.png" width="640" /></a></div><br /><o:p></o:p><p></p>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step III: Browse on Auth field and fill up the
credentials</span></h3>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Use the credentials used to log into F5 BIG-IP (in
this case, recently created new user)<o:p></o:p></span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhhTiDlnDqJUp_R2VMZAy6MtIRg8RjUi5YstauWYRougT6njB8RMwlWX8SrGytl_Njl7cDCx4QuhtiHriSul0jVhqdTVbOKEXvQVh2-yTp5Qy46hoQyaV0avIDfatMsY94-TRkbyqyF08bpnVUAmx_bcyAFN9RBQyUq2gnN-4VAGbkwib1jxyB7Yu9i/s1282/new%20PostMan%20second.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="756" data-original-width="1282" height="378" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhhTiDlnDqJUp_R2VMZAy6MtIRg8RjUi5YstauWYRougT6njB8RMwlWX8SrGytl_Njl7cDCx4QuhtiHriSul0jVhqdTVbOKEXvQVh2-yTp5Qy46hoQyaV0avIDfatMsY94-TRkbyqyF08bpnVUAmx_bcyAFN9RBQyUq2gnN-4VAGbkwib1jxyB7Yu9i/w640-h378/new%20PostMan%20second.png" width="640" /></a></div><br /><p class="MsoNormal"><br /></p>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step IV: Select on Body option </span></h3>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Change the method into POST, then select raw
sub-option and then JSON data format. Past the Telemetry Streaming declaration
on the body section and then click on the send button.<o:p></o:p></span></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;"><br /></span></p><p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">{<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "class": "Telemetry",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "My_Poller": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_System_Poller",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "interval": 0<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; },<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "My_System": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_System",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "enable": "true",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "systemPoller": [<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "My_Poller"<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ]<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; },<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "metrics": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_Pull_Consumer",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "type":
  "Prometheus",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "systemPoller":
  "My_Poller"<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; }<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">}<o:p></o:p></span></p><p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;"><br /></span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgS5j4JvPh67dRGxEhksgqcuGMYUtYkxdKb_r4T_pTRiKB1EEsOwMjaW5tsJ9NsyvYFxc0wTR9WN2FwYB1PGmyD4ZmX0-yRhwXkWfWQX3qJEInarSAZ7_GmVfFzZyWNJ2yGkZm2Z-cveap4rM2ShhABhAwAj9crPtfsnST42pVGAr1MtRQcpSXC6Bzy/s1283/new%20PostMan%20fourth.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="663" data-original-width="1283" height="330" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgS5j4JvPh67dRGxEhksgqcuGMYUtYkxdKb_r4T_pTRiKB1EEsOwMjaW5tsJ9NsyvYFxc0wTR9WN2FwYB1PGmyD4ZmX0-yRhwXkWfWQX3qJEInarSAZ7_GmVfFzZyWNJ2yGkZm2Z-cveap4rM2ShhABhAwAj9crPtfsnST42pVGAr1MtRQcpSXC6Bzy/w640-h330/new%20PostMan%20fourth.png" width="640" /></a></div><p class="MsoNormal"><br /></p>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step V: Verify the response as the success status</span><br /></h3>



<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwbaLj0lqgVe5KSGBz36cQdSAauibDrtT_nOTo-0jJWD5oaAmnrBOZNW0eNhKt2rEsx1g440QnJ5jFVYrKZYCuAv93iF3UZJR6cgX-TQOMZZZLU-U8T9dhhueXddsadLKGjp-f9eJvF8K6ogbh8pV4EWs1H8Sypj8wnyS0ajwHVvlRfcMctGwdDwcO/s1278/new%20PostMan%20third%20-%20response.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="872" data-original-width="1278" height="436" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwbaLj0lqgVe5KSGBz36cQdSAauibDrtT_nOTo-0jJWD5oaAmnrBOZNW0eNhKt2rEsx1g440QnJ5jFVYrKZYCuAv93iF3UZJR6cgX-TQOMZZZLU-U8T9dhhueXddsadLKGjp-f9eJvF8K6ogbh8pV4EWs1H8Sypj8wnyS0ajwHVvlRfcMctGwdDwcO/w640-h436/new%20PostMan%20third%20-%20response.png" width="640" /></a></div><br /><p></p>

<h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /></span></h3><h3 style="text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Step VI: Verify the available metrics </span></h3>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Create a new tab on Postman:<o:p></o:p></span></p>

<p class="MsoListParagraphCxSpFirst" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">-<span style="font-family: &quot;Times New Roman&quot;; font-size: 7pt; font-stretch: normal; font-variant-east-asian: normal; font-variant-numeric: normal; line-height: normal;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</span></span><!--[endif]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">On the URL section<o:p></o:p></span></p>

<p class="MsoListParagraphCxSpMiddle"><span class="MsoHyperlink"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">https://&lt;big-ip-management-ip-address&gt;/mgmt/shared/telemetry/pullconsumer/metrics</span></span><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><o:p></o:p></span></p>

<p class="MsoListParagraphCxSpLast" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">-<span style="font-family: &quot;Times New Roman&quot;; font-size: 7pt; font-stretch: normal; font-variant-east-asian: normal; font-variant-numeric: normal; line-height: normal;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
</span></span><!--[endif]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">On the authorization section, use the same
credentials used before<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhQ2r2E2OsaYz3bL-p_eMHF6uL7PhfvBuwOC2omCckBIS98et2BR3GXNIgztB2XcpOaEb1i-yCO9CY4osOf9U505bpr4_iQG0wlfWKw88xwPY4l26PHiiP-OlOqbwPUpzIbSet19VS4ZDzvTgp_SostH8BY6zPUbJKhKhKX4DofFkmQZQ9njVgv3s5t/s1279/new%20PostMan%20fourth%20verifying.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="265" data-original-width="1279" height="132" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhQ2r2E2OsaYz3bL-p_eMHF6uL7PhfvBuwOC2omCckBIS98et2BR3GXNIgztB2XcpOaEb1i-yCO9CY4osOf9U505bpr4_iQG0wlfWKw88xwPY4l26PHiiP-OlOqbwPUpzIbSet19VS4ZDzvTgp_SostH8BY6zPUbJKhKhKX4DofFkmQZQ9njVgv3s5t/w640-h132/new%20PostMan%20fourth%20verifying.png" width="640" /></a></div><p></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /></span></p><p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">You will obtain a response something like this:</span></p><p class="MsoNormal"></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgAdxcIWkME9WdrFl3UBELCI6F6dnFIDkdvJt7NvMSpSR6ArYY6aHHwuZ0R57wQVTKf1k9ELk7nLjy5ZoZRLY4x5NoQY_F532JWXF-H3PMnYrEGqTglMYEmMlLCZT__h-Jqrd3bW79TYJ8r9BxYV2chsnneRqljmQdVzY2c4DIx-8f8XZ01CwVAbeqE/s1283/new%20PostMan%20fifth%20veriying.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="874" data-original-width="1283" height="436" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgAdxcIWkME9WdrFl3UBELCI6F6dnFIDkdvJt7NvMSpSR6ArYY6aHHwuZ0R57wQVTKf1k9ELk7nLjy5ZoZRLY4x5NoQY_F532JWXF-H3PMnYrEGqTglMYEmMlLCZT__h-Jqrd3bW79TYJ8r9BxYV2chsnneRqljmQdVzY2c4DIx-8f8XZ01CwVAbeqE/w640-h436/new%20PostMan%20fifth%20veriying.png" width="640" /></a></div><br /><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /></span><p></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<h2 style="text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 14pt; line-height: 107%; mso-bidi-font-size: 11.0pt;">Configuration using CLI
of F5 BIG-IP device</span></h2>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Following steps for the
configuration of telemetry streaming consumer target using CLI of F5 BIG-IP
device are discussed below:<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Once you have accessed your F5 BIG-IP device CLI
terminal then access either your default admin credentials or the new user
you’ve recently created on the above section. Then execute the following
commands on the terminal:<o:p></o:p></span></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">On the username and password section, you either
enter your default admin credentials or the new user you’ve recently created
has the administrator privilege.<o:p></o:p></span></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: -0.25pt; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.75pt;" valign="top" width="624">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">curl -u username:password -k
  https://localhost/mgmt/shared/telemetry/declare <o:p></o:p></span></p>
  <p class="MsoListParagraph" style="line-height: normal; margin: 0in; mso-add-space: auto;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>Note: <o:p></o:p></b></span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>-k, --insecure <o:p></o:p></b></span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><b>to be made secure by using the CA
certificate bundle installed by default. This makes all connections considered
"insecure" fail unless -k, --insecure is used.</b><o:p></o:p></span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj1tezdKWpgrS5t0nD5OxM6SW3ShiwMSDoeI6w7hlyEelnWxFNtA1W-k8ypF7P5_eNj7oV2sDoGW7MBKz81iCw3DQSWLefG7NtJHbAJHeKvv8g--v8ua2q4DFPvEWms3ucpzXNnE4DP9NkeodY9OLHShgPMIqQCKIdTu1xWFHaCBYiQMDpsWMSgr_eT/s1482/CLI%20of%20F5%20first.png" style="clear: left; float: left; margin-bottom: 1em; margin-left: 1em;"><img border="0" data-original-height="210" data-original-width="1482" height="90" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj1tezdKWpgrS5t0nD5OxM6SW3ShiwMSDoeI6w7hlyEelnWxFNtA1W-k8ypF7P5_eNj7oV2sDoGW7MBKz81iCw3DQSWLefG7NtJHbAJHeKvv8g--v8ua2q4DFPvEWms3ucpzXNnE4DP9NkeodY9OLHShgPMIqQCKIdTu1xWFHaCBYiQMDpsWMSgr_eT/w640-h90/CLI%20of%20F5%20first.png" width="640" /></a></div><br /><p></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">ChangChange into tmp directory and
create a file called ts-config.json and I am using vi editor for it.&nbsp;&nbsp; <o:p></o:p></span></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">cd /tmp<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">vi ts-config.json<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgUTsztILYeYCUz5Bow-WOpP9MXVs8-KiN84WRPTAjB2ZU4FmQEH9Kstw5tUX-pbbs7DWsFdM3JM62lsL77ZP3xsEuknrWX8jMY-Ct8RyZIMdLYMB2qjvUxpasxD4HVgwVNqsS3_lzWRl4CrwqC0X5URAQksyr6lSmOQ0aJtuVhxuuEmMuWR7WfF2oW/s547/CLI%20of%20F5%20third.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="107" data-original-width="547" height="126" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgUTsztILYeYCUz5Bow-WOpP9MXVs8-KiN84WRPTAjB2ZU4FmQEH9Kstw5tUX-pbbs7DWsFdM3JM62lsL77ZP3xsEuknrWX8jMY-Ct8RyZIMdLYMB2qjvUxpasxD4HVgwVNqsS3_lzWRl4CrwqC0X5URAQksyr6lSmOQ0aJtuVhxuuEmMuWR7WfF2oW/w640-h126/CLI%20of%20F5%20third.png" width="640" /></a></div><br /><p></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Paste the Telemetry Streaming
declaration and then save the file and exit the vi editor.<o:p></o:p></span></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">{<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "class": "Telemetry",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "My_Poller": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_System_Poller",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "interval": 0<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; },<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "My_System": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_System",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "enable": "true",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "systemPoller": [<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "My_Poller"<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ]<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; },<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; "metrics": {<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "class":
  "Telemetry_Pull_Consumer",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "type":
  "Prometheus",<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "systemPoller":
  "My_Poller"<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; }<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">}<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgSudMYZ2-2jPG9M1-i9-E5La3fk9WfwmgZZSDxwGY4IUJtuFsOIjfINfy6irTtgNEk6SrWr2zUSQMrx0lXWNh1sdxmkDt7Th1jqRcsRJa3yW0pfnKoFXjKgBT24Aoe7xzQYUmX4dAFytMtVCy5UI-HvY0Z-NzGfEjF4heeVrLaABWshfIKjsW3vfwq/s662/CLI%20of%20F5%20second.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="428" data-original-width="662" height="414" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgSudMYZ2-2jPG9M1-i9-E5La3fk9WfwmgZZSDxwGY4IUJtuFsOIjfINfy6irTtgNEk6SrWr2zUSQMrx0lXWNh1sdxmkDt7Th1jqRcsRJa3yW0pfnKoFXjKgBT24Aoe7xzQYUmX4dAFytMtVCy5UI-HvY0Z-NzGfEjF4heeVrLaABWshfIKjsW3vfwq/w640-h414/CLI%20of%20F5%20second.png" width="640" /></a></div><br /><p></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Then execute the following
command on the terminal on the <b>same directory /tmp and change the username and
password section with your F5 BIG-IP device credentials</b> having the
administrator privilege.<o:p></o:p></span></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">curl -X POST -u
  username:password -k </span><a href="https://localhost/mgmt/shared/telemetry/declare"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">https://localhost/mgmt/shared/telemetry/declare</span></a><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;"> -d @ts-config.json -H
  “content-type:application/json”<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgMSY2ZTP8zTggP-GTQxFJbdQxPnrsx47JYelQk6hv9cJUG7W-K10spBg4M6qGKf9bP9AeNGtbfrhkkEtnx4ibb22FVpl7tjhx1QRHUzVGWKqDLfNBP-zhNWtgzNGQXOLIwRCPTPVmZWUkIUm_ybHUgGg2cVfce52fClLqwb26QbDSoXhv7s7CyLbXN/s1225/CLI%20of%20F5%20fourth.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="198" data-original-width="1225" height="104" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgMSY2ZTP8zTggP-GTQxFJbdQxPnrsx47JYelQk6hv9cJUG7W-K10spBg4M6qGKf9bP9AeNGtbfrhkkEtnx4ibb22FVpl7tjhx1QRHUzVGWKqDLfNBP-zhNWtgzNGQXOLIwRCPTPVmZWUkIUm_ybHUgGg2cVfce52fClLqwb26QbDSoXhv7s7CyLbXN/w640-h104/CLI%20of%20F5%20fourth.png" width="640" /></a></div><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span><div><br /><p></p>

<h3 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">To verify the available metrics</span></h3>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">curl -u username:password -k
  https://localhost/mgmt/shared/telemetry/pullconsumer/metrics<o:p></o:p></span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;<br /></span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgCQajveCxXd_CzvzU59o-CZ7yThC4v7YmhT4HMsiRUyKqDlxK2wUBFU2WOyyvWm_fAnrrci56S7lWwpCP9RPKRjhbxlH3jwXVISeUg_ljmYo0e7LewdlnsYXwyb0bTY6O5TAja8M0zpgcpd5AiAnroktYfV09Pb1jz85HCI3a0sP7VR36hHkD3K4nv/s1277/CLI%20of%20F5%20fifth.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="760" data-original-width="1277" height="380" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgCQajveCxXd_CzvzU59o-CZ7yThC4v7YmhT4HMsiRUyKqDlxK2wUBFU2WOyyvWm_fAnrrci56S7lWwpCP9RPKRjhbxlH3jwXVISeUg_ljmYo0e7LewdlnsYXwyb0bTY6O5TAja8M0zpgcpd5AiAnroktYfV09Pb1jz85HCI3a0sP7VR36hHkD3K4nv/w640-h380/CLI%20of%20F5%20fifth.png" width="640" /></a></div><br /><p></p>

<p class="MsoNormal"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<h2 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 12pt; line-height: 107%; mso-bidi-font-size: 10.0pt;">Section
III: Configuration of Prometheus </span></h2>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Once the telemetry streaming
service has been successfully configured and the metrics are available on the
path. We need to configure Prometheus in order to scrape the metrics data on
the predefined path. The following are the steps to configure the
Prometheus:<o:p></o:p></span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><i><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Note: On this user-guide
demonstration, both Grafana and Prometheus are installed on the same host with
different service ports as mentioned earlier. CentOS 7 is used as the OS for this
host machine and you may have different syntax to view the following status
check.<o:p></o:p></span></i></p>

<h3 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">First check the status of the
Prometheus</span></h3>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">sudo systemctl status
  prometheus.service<o:p></o:p></span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjkk1BhMGOKY0Zcvs-NF83o0C7RMbmTBu-bHFt8LX0fb4e62b1yWOEqXFz0Jq4MWBumnZu7PfFBmyBFGilFPRiySpVDiG5wT8m_Pjl-Amd73WEFmswuWFgSX0T6qjtS3pb90v4icoSO-QQTATOI6UYcHw8pNknG1GRYBwCDLzjUSY8z1VzDAiJJ-9zV/s1476/prometheus%20status%20checl.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="490" data-original-width="1476" height="213" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjkk1BhMGOKY0Zcvs-NF83o0C7RMbmTBu-bHFt8LX0fb4e62b1yWOEqXFz0Jq4MWBumnZu7PfFBmyBFGilFPRiySpVDiG5wT8m_Pjl-Amd73WEFmswuWFgSX0T6qjtS3pb90v4icoSO-QQTATOI6UYcHw8pNknG1GRYBwCDLzjUSY8z1VzDAiJJ-9zV/w640-h213/prometheus%20status%20checl.png" width="640" /></a></div><br /><p></p>

<h3 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">View the current working
directory and change into /etc/prometheus</span></h3>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">pwd<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">cd /etc/prometheus<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">ls -al<o:p></o:p></span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg9Oqk5xY7S8743oshw9DAKN6nIgs5NBSvoO9oBoSUVK6YwTXOVq-4e6T9FNfKABN0A6VR4LQ4BuS-fAGWwRGogD6jfsDlYBXUSh_gMssbYZEjxD9og4si_uDPbJM1p1CaujTWKHL5Ovmr7HZvJYXPztyE90tmzsnctYHW4OJM9VG35xQSEJyzU6frs/s719/prometheus%20cofig%20file.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="324" data-original-width="719" height="288" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg9Oqk5xY7S8743oshw9DAKN6nIgs5NBSvoO9oBoSUVK6YwTXOVq-4e6T9FNfKABN0A6VR4LQ4BuS-fAGWwRGogD6jfsDlYBXUSh_gMssbYZEjxD9og4si_uDPbJM1p1CaujTWKHL5Ovmr7HZvJYXPztyE90tmzsnctYHW4OJM9VG35xQSEJyzU6frs/w640-h288/prometheus%20cofig%20file.png" width="640" /></a></div><br /><p></p>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">global:<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp; scrape_interval: 10s<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">scrape_configs:<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp; - job_name: 'TelemetryStreaming'<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; scrape_timeout: 30s<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; scrape_interval: 30s<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; scheme: https<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; tls_config:<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp; insecure_skip_verify: true<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; metrics_path:
  '/mgmt/shared/telemetry/pullconsumer/metrics'<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; basic_auth:<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp; username: 'F5-BIG-IP-username'<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp; password: 'F5-BIG-IP-password'<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp; static_configs:<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - targets: ['BIGIP-managementIP:443']<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">&nbsp;</span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhr0RVBnWZnACaU3FKy4yHhyLqEN6Tiucwb_V77uD_NVSHBXoG8nA9FA0gviDT7q7tnq_kz6FDlYat_gKrHCQyGSD3DQta1hPpHvWppVOkYBzb9ly27zDHBcvp_b04Jv7R6C5KjB1ivPJWgLDRw2TO0NoOPOSBlLS6yDWw0oJmKDhSXBMP6m2kK9lod/s726/prometheus%20configuration%20for%20telemetry%20streaming.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="415" data-original-width="726" height="366" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhr0RVBnWZnACaU3FKy4yHhyLqEN6Tiucwb_V77uD_NVSHBXoG8nA9FA0gviDT7q7tnq_kz6FDlYat_gKrHCQyGSD3DQta1hPpHvWppVOkYBzb9ly27zDHBcvp_b04Jv7R6C5KjB1ivPJWgLDRw2TO0NoOPOSBlLS6yDWw0oJmKDhSXBMP6m2kK9lod/w640-h366/prometheus%20configuration%20for%20telemetry%20streaming.png" width="640" /></a></div><br /><div class="separator" style="clear: both; text-align: center;"><br /></div>

<h3 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Then restart the Prometheus
service and check the status of the Prometheus service.</span></h3>

<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 0.25in; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0in 5.4pt 0in 5.4pt; mso-yfti-tbllook: 1184;">
 <tbody><tr>
  <td style="border: 1pt solid windowtext; mso-border-alt: solid windowtext .5pt; padding: 0in 5.4pt; width: 467.5pt;" valign="top" width="623">
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">sudo systemctl restart
  prometheus.service<o:p></o:p></span></p>
  <p class="MsoNormal" style="line-height: normal; margin-bottom: 0in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt;">sudo systemctl status
  prometheus.service<o:p></o:p></span></p>
  </td>
 </tr>
</tbody></table>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhrca9MaBs8HBADaCcEaHn4yYK6PbJaHBAXWPytMNnyeJlxB0jIvTq-WGSH5OyIjsdCsnAZOP1EKQ1YZivazIw6NunKSY1Zsi4RyTUYoyGmPJNR4NikXhzgQgQeBiYAQGTAJsYpRQjrNmZzx3rlDHYBsoU7zOgJHc5l7f-2GvHuimJqmBs8Ll_1ZQwo/s1475/prometheus%20status.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="444" data-original-width="1475" height="192" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhrca9MaBs8HBADaCcEaHn4yYK6PbJaHBAXWPytMNnyeJlxB0jIvTq-WGSH5OyIjsdCsnAZOP1EKQ1YZivazIw6NunKSY1Zsi4RyTUYoyGmPJNR4NikXhzgQgQeBiYAQGTAJsYpRQjrNmZzx3rlDHYBsoU7zOgJHc5l7f-2GvHuimJqmBs8Ll_1ZQwo/w640-h192/prometheus%20status.png" width="640" /></a></div><br /><p></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Note: If the configuration is
correct, then the Prometheus service will be enabled otherwise, the status of the Prometheus service will be disabled.<o:p></o:p></span></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><br /></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<h3 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">To further verify whether instances has been discovered on the the
Prometheus:</span></h3>

<p class="MsoListParagraphCxSpFirst" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">-<span style="font-family: &quot;Times New Roman&quot;; font-size: 7pt; font-stretch: normal; font-variant-east-asian: normal; font-variant-numeric: normal; line-height: normal;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; </span></span></span><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Go to </span><a href="http://prometheus-ip:service/port"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">http://prometheus-ip:service/port</span></a><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><o:p></o:p></span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhYhFR8LKIl1yFUDSKF7xXC_W5eTtqzufMJ6KFFpqpndAYbUTEvrWM8DBTw1IltSf_f4LhjROyIZ_4aMcSIBnD8Ya6zqvi8Wp3nAl-pcHLFfVbM9wV14ROCPsqiT8IdKK8NJOya5-mr2R8t3a3DRWTfpL73ZXxlzHjItaCR8SBckf6Wl1C26P5QjDrT/s1809/prometheus%20dashboard.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="635" data-original-width="1809" height="224" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhYhFR8LKIl1yFUDSKF7xXC_W5eTtqzufMJ6KFFpqpndAYbUTEvrWM8DBTw1IltSf_f4LhjROyIZ_4aMcSIBnD8Ya6zqvi8Wp3nAl-pcHLFfVbM9wV14ROCPsqiT8IdKK8NJOya5-mr2R8t3a3DRWTfpL73ZXxlzHjItaCR8SBckf6Wl1C26P5QjDrT/w640-h224/prometheus%20dashboard.png" width="640" /></a></div><p class="MsoListParagraphCxSpMiddle"><br /></p>

<p class="MsoListParagraphCxSpMiddle"><br /></p>

<h4 style="text-align: left; text-indent: -0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">-<span style="font-family: &quot;Times New Roman&quot;; font-size: 7pt; font-stretch: normal; font-variant-east-asian: normal; font-variant-numeric: normal; line-height: normal;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span></span></span><!--[endif]--><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Click on the Status option and select the Target option<o:p></o:p></span></h4><p class="MsoListParagraphCxSpMiddle" style="mso-list: l0 level1 lfo1; text-indent: -0.25in;"><!--[if !supportLists]--></p>

<p class="MsoListParagraphCxSpMiddle"></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgmx0aBBd4ho76YMJGS8CJ04ZRx54UYb7rs95pzYW0g04fdXWXzu92wDAPuy0yVNuBlKtDegtiRbyGynwTZosIJvN_K49U1QF1hozMLC3SPJabD05XuV1vUHx-VO3CC0gGy68eTT_F1tAELRNSP2k6Q4Qapo_TH2LuR-jFi5RCP8GUDg3mFRUZymqF9/s1818/Prometheus%20instance%20status.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="552" data-original-width="1818" height="194" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgmx0aBBd4ho76YMJGS8CJ04ZRx54UYb7rs95pzYW0g04fdXWXzu92wDAPuy0yVNuBlKtDegtiRbyGynwTZosIJvN_K49U1QF1hozMLC3SPJabD05XuV1vUHx-VO3CC0gGy68eTT_F1tAELRNSP2k6Q4Qapo_TH2LuR-jFi5RCP8GUDg3mFRUZymqF9/w640-h194/Prometheus%20instance%20status.png" width="640" /></a></div><br /><p></p>

<p class="MsoListParagraphCxSpMiddle"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p><div class="separator" style="clear: both; text-align: center;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiIdvKy-6xTnqg32GSDsdPru4DGUcJSLj0HEItvvf85HGm5jQOSIYdrG8KBmC_JgAWnZGnGUaaraZp19uJ61SLFxG_VKTCYzVjQiNEuNalnA9IbkJ7tZEeHi-SqN2KYBgUAITovPo6EKlkzUVzo42fEfYL4QH9rMYPThGUNXf2r4sSH-2i18Zk_E9dV/s1819/prometheus%20metrics%20available.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1067" data-original-width="1819" height="376" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiIdvKy-6xTnqg32GSDsdPru4DGUcJSLj0HEItvvf85HGm5jQOSIYdrG8KBmC_JgAWnZGnGUaaraZp19uJ61SLFxG_VKTCYzVjQiNEuNalnA9IbkJ7tZEeHi-SqN2KYBgUAITovPo6EKlkzUVzo42fEfYL4QH9rMYPThGUNXf2r4sSH-2i18Zk_E9dV/w640-h376/prometheus%20metrics%20available.png" width="640" /></a></span></div><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;"><br /><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgEsyB86RIa8NTTOExvFyKz_OBHavmftzUnO0aSOZkTrFAt1d8FXGyuxt9OQ7nXaMJtv3GLtByGkI-JxcjBJL2Y5vCs2S6jJqLDr4qpl0NJeRIKz-azpXwX_ldXwuU1KzBH_hhxQOm9WI1ULJy4H-VYlqf84YDLVtILEfBGiPZqPCrSA8KAkf5lvZ_Q/s1797/prometheus%20service%20checl.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1080" data-original-width="1797" height="384" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgEsyB86RIa8NTTOExvFyKz_OBHavmftzUnO0aSOZkTrFAt1d8FXGyuxt9OQ7nXaMJtv3GLtByGkI-JxcjBJL2Y5vCs2S6jJqLDr4qpl0NJeRIKz-azpXwX_ldXwuU1KzBH_hhxQOm9WI1ULJy4H-VYlqf84YDLVtILEfBGiPZqPCrSA8KAkf5lvZ_Q/w640-h384/prometheus%20service%20checl.png" width="640" /></a></div></span><p></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">&nbsp;</span></p>

<h2 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 12pt; line-height: 107%; mso-bidi-font-size: 10.0pt;"><br /></span></h2><h2 style="margin-left: 0.25in; text-align: left;"><span face="&quot;Arial Narrow&quot;,sans-serif" style="font-size: 12pt; line-height: 107%; mso-bidi-font-size: 10.0pt;">Section
IV: Configuration on Grafana using Prometheus as a data source</span></h2>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">In this section, we need to c</span><span style="font-size: 12pt;">onnect Prometheus as a data source
on Grafana</span></p>

<div class="separator" style="clear: both; text-align: center;"><br /></div><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiHiv5RRNwyBaq1wHEcxHVjG3-aQ6WtTpo1I-LzhagsJzAkgKETEnXO13DXdB5mifMPK5-b88k1xCBzPAyoLPUYaJySeYAVSkKqddFifvt081BcuN2zmYc98-F4cJmGsAJu1AoIpfUETg5dvCZsoKU_SjiTxn6t82IvkADYQ894zrBieyy18sfdCJeo/s1820/grafana%20configration.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1080" data-original-width="1820" height="380" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiHiv5RRNwyBaq1wHEcxHVjG3-aQ6WtTpo1I-LzhagsJzAkgKETEnXO13DXdB5mifMPK5-b88k1xCBzPAyoLPUYaJySeYAVSkKqddFifvt081BcuN2zmYc98-F4cJmGsAJu1AoIpfUETg5dvCZsoKU_SjiTxn6t82IvkADYQ894zrBieyy18sfdCJeo/w640-h380/grafana%20configration.png" width="640" /></a></div><div class="separator" style="clear: both; text-align: center;"><br /></div><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiOq4Mw40e5Bt1Czp3KoFt_4JBv18TMdd397XHKqaPEVuTlcA6oo-bvbW_gBFTcz1_12vqPKM1mZKNBmiGh4LC6nBnhrx8ZQfGKjNsiDi-Qvniv8pgr8ynh-ne9CcQ6ijY27WS1ZAX1oh8xjApJRB6b6eynKKjQ_mddVpqmvWhkvwNY2gvvk5BLt_1x/s1822/localhost%20on%20grafana%20(new).png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="952" data-original-width="1822" height="334" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiOq4Mw40e5Bt1Czp3KoFt_4JBv18TMdd397XHKqaPEVuTlcA6oo-bvbW_gBFTcz1_12vqPKM1mZKNBmiGh4LC6nBnhrx8ZQfGKjNsiDi-Qvniv8pgr8ynh-ne9CcQ6ijY27WS1ZAX1oh8xjApJRB6b6eynKKjQ_mddVpqmvWhkvwNY2gvvk5BLt_1x/w640-h334/localhost%20on%20grafana%20(new).png" width="640" /></a></div><br /><div class="separator" style="clear: both; text-align: center;"><br /></div><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjuadtqOFJ3bzb89lQSSWaZGHUBeWPy4n_v6Xzenk-907RTVcr2sfmM2LXjNdrlrWz-268sGFOF46JuaQVQoFOukKxJpGPZa5zRBKBerIh76zBG3Aldof0bynGtQfQkAsHQQoJ5KCv_1Ll2FSdFVpDxH0WJ0AZASimNONUV4r3YYAZw3yE20UUuEWXC/s1825/successfully%20added%20prometheus%20as%20data%20source.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1012" data-original-width="1825" height="354" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjuadtqOFJ3bzb89lQSSWaZGHUBeWPy4n_v6Xzenk-907RTVcr2sfmM2LXjNdrlrWz-268sGFOF46JuaQVQoFOukKxJpGPZa5zRBKBerIh76zBG3Aldof0bynGtQfQkAsHQQoJ5KCv_1Ll2FSdFVpDxH0WJ0AZASimNONUV4r3YYAZw3yE20UUuEWXC/w640-h354/successfully%20added%20prometheus%20as%20data%20source.png" width="640" /></a></div><p class="MsoNormal" style="margin-left: 0.25in;"><br /></p>

<p class="MsoNormal" style="margin-left: 0.25in;"><span face="&quot;Arial Narrow&quot;, sans-serif" style="font-size: 12pt; line-height: 107%;">Once the data source has been successfully configured on Grafna then Create a new dashboard and select
Prometheus as the data source then select the relevant metrics and change the
refresh interval as required. Save and apply the panel. Then,&nbsp;<o:p></o:p></span><span style="font-size: 16px;">Save the dashboard and view the metrics on the Grafana dashboard.</span></p>

<div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEghvn9K3aXnVYVtUElSoUe-A8oUzCrUV-izIWh_s9ENkmf1SnCdrY5qSuxVX5NEttDZrGpITfRQQ_81lG7j-0EI49RUbmhaxarM3aMm4lP-_2tpH6R10-QNDaCpwSlGNh6Psaeoi6Bhh5YceYoprK1IcYBjwGyq0RXYwbSa1wz84csXdBHssCZOBEVU/s1821/configuration%20of%20grafana%20using%20prometheus%20metrics.png" style="margin-left: 1em; margin-right: 1em;"></a><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEghvn9K3aXnVYVtUElSoUe-A8oUzCrUV-izIWh_s9ENkmf1SnCdrY5qSuxVX5NEttDZrGpITfRQQ_81lG7j-0EI49RUbmhaxarM3aMm4lP-_2tpH6R10-QNDaCpwSlGNh6Psaeoi6Bhh5YceYoprK1IcYBjwGyq0RXYwbSa1wz84csXdBHssCZOBEVU/s1821/configuration%20of%20grafana%20using%20prometheus%20metrics.png" style="margin-left: 1em; margin-right: 1em;"></a><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiaBqf5cMPM1wEfac-6lSASUC3rv-fwvxOVQlIb4cnPF0J2eKDMRCyRVGsan5EdO1whpVdv7nbIqzFNYCvzrRMdJE-kaqRP_sw8XyW_sAt9Fpj6qDwbmdyuXrfUqnvCY3sDxmv1BMmm_BGX3-eeRIoOxRCh1kMy74GHDzGqa7sHhdABCGIrivCuft28/s1824/adding%20new%20panel%20on%20Grafana.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="727" data-original-width="1824" height="239" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiaBqf5cMPM1wEfac-6lSASUC3rv-fwvxOVQlIb4cnPF0J2eKDMRCyRVGsan5EdO1whpVdv7nbIqzFNYCvzrRMdJE-kaqRP_sw8XyW_sAt9Fpj6qDwbmdyuXrfUqnvCY3sDxmv1BMmm_BGX3-eeRIoOxRCh1kMy74GHDzGqa7sHhdABCGIrivCuft28/w598-h239/adding%20new%20panel%20on%20Grafana.png" width="598" /></a></div><div class="separator" style="clear: both; text-align: center;"><br /></div><img border="0" data-original-height="1056" data-original-width="1821" height="336" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEghvn9K3aXnVYVtUElSoUe-A8oUzCrUV-izIWh_s9ENkmf1SnCdrY5qSuxVX5NEttDZrGpITfRQQ_81lG7j-0EI49RUbmhaxarM3aMm4lP-_2tpH6R10-QNDaCpwSlGNh6Psaeoi6Bhh5YceYoprK1IcYBjwGyq0RXYwbSa1wz84csXdBHssCZOBEVU/w605-h336/configuration%20of%20grafana%20using%20prometheus%20metrics.png" width="605" /></div>

<p class="MsoNormal" style="margin-left: 0.25in;"><br /></p><div class="separator" style="clear: both; text-align: center;"><a href="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEis-9OdRn7zf5DvDd1dQ3qh2EX5TLGabUpzLmilI7I5TcKk0WaEOV5jIN-SSZ0uvIe_WyttvNbSUD7F-exBeN2tVFv-6HP-A0ywYeuq8jPTOjT27vBouL9ltZwVFvAnIgt54WVnM0Q5dA7qEraqjkQOJDVbXVDzFjZhM6YVw_46P7Nf0hTPCi0VG0TQ/s1823/panel%20save%20on%20grafana.png" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="666" data-original-width="1823" height="234" src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEis-9OdRn7zf5DvDd1dQ3qh2EX5TLGabUpzLmilI7I5TcKk0WaEOV5jIN-SSZ0uvIe_WyttvNbSUD7F-exBeN2tVFv-6HP-A0ywYeuq8jPTOjT27vBouL9ltZwVFvAnIgt54WVnM0Q5dA7qEraqjkQOJDVbXVDzFjZhM6YVw_46P7Nf0hTPCi0VG0TQ/w640-h234/panel%20save%20on%20grafana.png" width="640" /></a></div><br /></div><div><br /></div><h2 style="text-align: left;">The possible issue that can arise during the configuration</h2><div>If you use the default TS declare from the official telemetry streaming document website then you may fail to view the available metrics on the mentioned link:</div><div>https://&lt;f5-management-ip&gt;/mgmt/shared/telemetry/pullconsumer/metrics</div>
