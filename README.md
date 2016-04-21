# aludratest-jenkins-plugin

Jenkins Plugin providing advanced statistics and charts for AludraTest executions.

## Description

This plugin helps you overcoming the limited standard test execution reporting of Jenkins and its standard plugins.

It is tailor-made for AludraTest executions and adds the "builds" dimension to your AludraTest reporting. 

![image](https://cloud.githubusercontent.com/assets/8896947/14713784/c4a4d300-07e2-11e6-9038-ad902ed0ddf9.png)

Discover how test cases run over several builds, quickly identifying potentially unstable test cases (or, instabilities in your Application Under Test!).

![image](https://cloud.githubusercontent.com/assets/8896947/14713902/454b42aa-07e3-11e6-9653-c1825aed6071.png)

*A click on each point opens the full log for exactly this test case execution, showing in detail what went wrong.*


Compare results of one build to another, which helps you identifying trends.

![image](https://cloud.githubusercontent.com/assets/8896947/14714032/bbec7f5a-07e3-11e6-9951-19d50d600e7a.png)


## Installation

Currently, this Plugin is not available on the central Jenkins Repository. Clone the project locally, run 

    mvn clean package

to generate an `aludratest.hpi` in the `target` folder. Open up your Jenkins Plugin Manager, switch to "Advanced", and upload the generated HPI file. Restart Jenkins if required.

## Configuration

To generate AludraTest statistics suitable for the AludraTest Jenkins Plugin, you have to perform two to three steps:

* Configure your log4testing.xml to write XML and HTML log
* Configure your Jenkins job to archive HTML log (optional)
* Add the Post-Build Step for generating the AludraTest report, and configure it.

The AludraTest Jenkins Plugin's Post Build Step "Gather AludraTest Statistics" collects information about test execution from an XML file written by log4testing. For HTML logs being available, you usually also configure HTML files to be written by log4testing. An example log4testing.xml can look this way:

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration xmlns="http://aludratest.org/log4testing/1.0">

		<!-- global properties - configure output base path here -->
		<properties>
			<property>
				<key>log4testing_outputdir</key>
				<value>target/result</value>
			</property>
		</properties>

		<writers>
			<!-- HTML writer without check() calls -->
			<writer>
				<class>org.aludratest.log4testing.html.HtmlTestLogWriter</class>
				<properties>
					<property>
						<key>outputFolder</key>
						<value>${log4testing_outputdir}/html</value>
					</property>
				</properties>
				
				<testStepFilters>
					<testStepFilter>
						<class>org.aludratest.log4testing.filter.RegExpCommandFilter</class>
						<properties>
							<property>
								<key>commandRegexp</key>
								<value>^(?!(is|has)).*</value>
							</property>
						</properties>
					</testStepFilter>
				</testStepFilters>
			</writer>
			
			<!-- XML writer for Jenkins plugin -->
			<writer>
				<class>org.aludratest.log4testing.xml.XmlTestLogWriter</class>
				<properties>
					<property>
						<key>outputFile</key>
						<value>${log4testing_outputdir}/result.xml</value>
					</property>
					<property>
						<key>compress</key>
						<value>false</value>
					</property>
				</properties>
				
				<!-- we do not need the step details for Jenkins plugin -->
				<testStepFilters>
					<testStepFilter>
						<class>org.aludratest.log4testing.filter.NoTestStepsFilter</class>
					</testStepFilter>
				</testStepFilters>
			</writer>
		</writers>
	</configuration>

Put this `log4testing.xml` in your `src/main/resources` directory, so it is consumed by AludraTest during execution. After execution, a file `target/result/result.xml` should have been generated, along with several HTML files in `target/result/html`.

Now, configure your Jenkins job with one or two post-build steps: First, you could add an "Archive artifacts" step which archives everything in `target/result/html`. We strongly recommend this to have HTML log links available from AludraTest Plugin's charts and tables.

Then, add the Post-Build Step "Generate AludraTest Report". Configure it to use `target/result/result.xml` as the result XML file, and to look within `artifact/target/result/html` for HTML logs.

Now, after a new execution of the job, the execution statistics will be gathered and a new item becomes available in the project's menu on the left side:

![image](https://cloud.githubusercontent.com/assets/8896947/14716700/7c236626-07ee-11e6-9b23-7e3fb2095048.png)

## Aggregating multiple projects

-Documentation coming soon-
