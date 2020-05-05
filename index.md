<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <title>Cloud Native | Nutanix&trade; Workshops</title>
        <meta name="robots" content="noindex, nofollow">
        <link rel="stylesheet" media="all" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" media="all" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/font-awesome/css/font-awesome.min.css">
        <link rel="stylesheet" media="screen" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/workshops/css/screen.css">
        <link rel="stylesheet" media="print" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/workshops/css/print.css">

        <link rel="shortcut icon" href="https://s3.amazonaws.com/handsonworkshops.prod.media/a/a/logo/nutanix-favicon.ico">

    </head>
<h1 id="nutanix-calm-and-karbon-cicd">Nutanix Calm and Karbon: CI/CD</h1>
<p><em>The estimated time to complete this lab is 60 minutes.</em></p>
<h2 id="overview">Overview</h2>
<p>Countless studies have shown that reducing the amount of time for developers to receive feedback on code changes improves software quality. Automating the build, test, and deployment of software with tools such as Jenkins is one of the best ways to accelerate software development. If you’re unfamiliar with Jenkins you can learn more on their website <a href="https://www.cloudbees.com/jenkins/about">here</a>.</p>
<p>In this lab, we'll utilize Nutanix Calm to build the infrastructure required to create a Continuous Integration / Continuous Delivery (CI/CD) Pipeline, which includes deploying Jenkins, a <a href="https://gitea.io/en-us/">Gitea</a> git server, a developer workstation, and configuring that workstation to manage a Karbon Kubernetes cluster. Once the infrastructure is deployed via Nutanix Calm, we'll go through configuring Jenkins and Gitea to create a fully function CI/CD pipeline. The end result will be a developer running a &quot;git push&quot;, which triggers Jenkins to build a docker container based on the new code, publishes that container to DockerHub, and then deploys that container onto the Nutanix Karbon Kubernetes cluster.</p>
<p><img src="images/01_cicd_pipeline.png" /></p>
<h2 id="pre-requisites">Pre-requisites</h2>
<p>This lab requires:</p>
<ul>
<li>a <strong>running Nutanix Karbon Kubernetes cluster</strong>. When the Calm blueprint is launched in the next step, a runtime variable will prompt you for the Karbon Kubernetes cluster name. If this bootcamp environment was staged, then use the default value of <strong>karbon_bootcamp_cluster</strong>. Otherwise, please specify the name of an <em>existing</em> Karbon Kubernetes cluster.</li>
<li>a <strong>DockerHub account</strong>. If you do not already have one, go ahead and sign up for an <a href="https://hub.docker.com/">account</a> (it's free).</li>
</ul>
<p>All other required components are deployed via a Calm blueprint.</p>
<h2 id="creating-our-cicd-infrastructure-with-nutanix-calm">Creating our CI/CD Infrastructure with Nutanix Calm</h2>
<p>Building a CI/CD pipeline generally involves connecting a large number of disparate tools. To streamline this lab, we'll be deploying many of these tools via a Nutanix Calm blueprint. We'll first launch the blueprint, and then in the ~20 minutes it takes to deploy, we'll cover the blueprint architecture to familiarze ourselves with the tools involved.</p>
<ol>
<li>In <strong>Prism Central</strong>, select bars <strong>&gt; Services &gt; Calm</strong>.</li>
<li><p>Select <img src="../images/blueprints_icon.png" alt="bp-icon" /> <strong>Blueprints</strong> in the left hand toolbar to view and manage Calm blueprints.</p>
<blockquote>
<p><strong>note</strong></p>
<p>Mousing over an icon will display its title.</p>
</blockquote></li>
</ol>
<ol>
<li>Select the blueprint titled <strong>CICD_Infra</strong>.</li>
<li><p>Click <strong>Credentials</strong> plus-circle and fill out the following fields:</p>
<ul>
<li><strong>Credential Name</strong> - CENTOS</li>
<li><strong>Username</strong> - centos</li>
<li><strong>Secret Type</strong> - SSH Private Key</li>
<li><strong>Key</strong> - Paste in your own private key, or use:</li>
</ul>
<pre><code>-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAii7qFDhVadLx5lULAG/ooCUTA/ATSmXbArs+GdHxbUWd/bNG
ZCXnaQ2L1mSVVGDxfTbSaTJ3En3tVlMtD2RjZPdhqWESCaoj2kXLYSiNDS9qz3SK
6h822je/f9O9CzCTrw2XGhnDVwmNraUvO5wmQObCDthTXc72PcBOd6oa4ENsnuY9
HtiETg29TZXgCYPFXipLBHSZYkBmGgccAeY9dq5ywiywBJLuoSovXkkRJk3cd7Gy
hCRIwYzqfdgSmiAMYgJLrz/UuLxatPqXts2D8v1xqR9EPNZNzgd4QHK4of1lqsNR
uz2SxkwqLcXSw0mGcAL8mIwVpzhPzwmENC5OrwIBJQKCAQB++q2WCkCmbtByyrAp
6ktiukjTL6MGGGhjX/PgYA5IvINX1SvtU0NZnb7FAntiSz7GFrODQyFPQ0jL3bq0
MrwzRDA6x+cPzMb/7RvBEIGdadfFjbAVaMqfAsul5SpBokKFLxU6lDb2CMdhS67c
1K2Hv0qKLpHL0vAdEZQ2nFAMWETvVMzl0o1dQmyGzA0GTY8VYdCRsUbwNgvFMvBj
8T/svzjpASDifa7IXlGaLrXfCH584zt7y+qjJ05O1G0NFslQ9n2wi7F93N8rHxgl
JDE4OhfyaDyLL1UdBlBpjYPSUbX7D5NExLggWEVFEwx4JRaK6+aDdFDKbSBIidHf
h45NAoGBANjANRKLBtcxmW4foK5ILTuFkOaowqj+2AIgT1ezCVpErHDFg0bkuvDk
QVdsAJRX5//luSO30dI0OWWGjgmIUXD7iej0sjAPJjRAv8ai+MYyaLfkdqv1Oj5c
oDC3KjmSdXTuWSYNvarsW+Uf2v7zlZlWesTnpV6gkZH3tX86iuiZAoGBAKM0mKX0
EjFkJH65Ym7gIED2CUyuFqq4WsCUD2RakpYZyIBKZGr8MRni3I4z6Hqm+rxVW6Dj
uFGQe5GhgPvO23UG1Y6nm0VkYgZq81TraZc/oMzignSC95w7OsLaLn6qp32Fje1M
Ez2Yn0T3dDcu1twY8OoDuvWx5LFMJ3NoRJaHAoGBAJ4rZP+xj17DVElxBo0EPK7k
7TKygDYhwDjnJSRSN0HfFg0agmQqXucjGuzEbyAkeN1Um9vLU+xrTHqEyIN/Jqxk
hztKxzfTtBhK7M84p7M5iq+0jfMau8ykdOVHZAB/odHeXLrnbrr/gVQsAKw1NdDC
kPCNXP/c9JrzB+c4juEVAoGBAJGPxmp/vTL4c5OebIxnCAKWP6VBUnyWliFhdYME
rECvNkjoZ2ZWjKhijVw8Il+OAjlFNgwJXzP9Z0qJIAMuHa2QeUfhmFKlo4ku9LOF
2rdUbNJpKD5m+IRsLX1az4W6zLwPVRHp56WjzFJEfGiRjzMBfOxkMSBSjbLjDm3Z
iUf7AoGBALjvtjapDwlEa5/CFvzOVGFq4L/OJTBEBGx/SA4HUc3TFTtlY2hvTDPZ
dQr/JBzLBUjCOBVuUuH3uW7hGhW+DnlzrfbfJATaRR8Ht6VU651T+Gbrr8EqNpCP
gmznERCNf9Kaxl/hlyV5dZBe/2LIK+/jLGNu9EJLoraaCBFshJKF
-----END RSA PRIVATE KEY-----</code></pre>
<p><img src="images/4.png" /></p></li>
<li>Click <strong>Save</strong>, and then <strong>Back</strong>.</li>
<li>Update the <strong>JenkinsAHVMaster</strong>. <strong>JenkinsAHVSlave</strong>, <strong>Workstation_AHV</strong>, and <strong>Gitea_AHV</strong> services with the following information:
<ul>
<li><strong>NIC 1</strong> - Primary</li>
</ul></li>
<li>Click <strong>Save</strong></li>
<li>Click <strong>Launch</strong></li>
<li><p>On the launch page, fill in the following fields.</p>
<ul>
<li><strong>Name of the Application</strong> - <em>initials</em>-cicd-infra</li>
<li><strong>gitea_password</strong> - Any password desired, which will be set as the Gitea admin user password</li>
<li><strong>karbon_cluster_name</strong> - The name of the Karbon cluster to use for this lab (it <strong>must</strong> already be depoyed). If the cluster was staged, leave the default of <strong>karbon_bootcamp_cluster</strong>.</li>
</ul>
<p><img src="images/03_bp_launch_2.png%0A%20:align:%20center%0A%20:alt:%20Nutanix%20Calm%20CI/CD%20Infrastructure%20Blueprint%20Launch%202" /></p></li>
<li>Click the blue <strong>Create</strong> button, and ensure you're redirected to the application page.</li>
</ol>
<p>Now that we're waiting for our CI/CD Infrastructure to deploy, let's review the architecture of the blueprint. If desired, open the blueprint in Calm and view the Services and their underlying scripts as they're covered. Alternatively, here's an image of the blueprint canvas.</p>
<p><img src="images/04_calm_cicd_infra_bp.png%0A%20:align:%20center%0A%20:alt:%20Nutanix%20Calm%20CI/CD%20Infrastructure%20Blueprint%20Architecture" /></p>
<p>In approximate order (approximate as Calm deploys Services in parallel, unless there is a dependency), the blueprint deploys the following Services:</p>
<ul>
<li><strong>Kubernetes</strong> (Existing machine) - this Service utilizes VM Pre-create eScript tasks to make API calls into Prism Central, to find a Karbon Kubernetes cluster matching the <strong>karbon_cluster_name</strong> variable defined at launch. It also sets the content of the cluster's <a href="https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/">kubeconfig</a> as a variable, which will be later applied to the Workstation VM.</li>
<li><strong>Gitea</strong> (AHV) - this Service installs Gitea, which is a community managed lightweight code hosting solution. It first installs MySQL, as Gitea requires a backend DB to operate. It then creates self signed certificates, installs the Gitea service, and configures the repo which stores our application code.</li>
<li><strong>Jenkins_Master</strong> (AHV) - this Service installs Jenkins, a popular Continuous Integration server. It also trusts Certificate Authority (CA) generated during the Package Install of the <strong>Gitea</strong> Service.</li>
<li><strong>Jenkins_Slave</strong> (AHV) - this Service installs a Jenkins Slave, which is used for builds in our Jenkins pipeline. Meaning this is the node that is responsible for building a docker container based on the new application code, publishing tha container to DockerHub, and then deploying the new container to the Kubernetes cluster.</li>
<li><strong>Workstation</strong> (AHV) - this Service represents a &quot;developer workstation,&quot; and is where we'll be making changes to our application code later in this lab. It first installs necessary software (like <em>git</em> and <em>kubectl</em>), and then configures the kubeconfig based on the variable set in the <strong>Kubernetes</strong> Service. Finally, it clones the git repo configured in the <strong>Gitea</strong> Service.</li>
</ul>
<p>Once your Jenkins_Master and Jenkins_Slave Services have been fully deployed, move on to the next section.</p>
<h2 id="accessing-jenkins">Accessing Jenkins</h2>
<p>Now that our CI/CD Infrastructure has been deployed, we're ready to start configuration of the various components that make up our Pipeline. First up, is Jenkins.</p>
<ol>
<li><p>On the <strong>Overview</strong> tab of your Application, <strong>right click</strong> on the <strong>Jenkins</strong> link, and open the page in a new tab.</p>
<p><img src="images/05_app_overview.png%0A%20:align:%20center%0A%20:alt:%20Nutanix%20Calm%20CI/CD%20Infrastructure%20App%20Overview" /></p></li>
<li><p>While still within the Calm Application page, navigate to the <strong>Services</strong> tab, select the <strong>Jenkins_Master</strong> Service, and in the right column, click <strong>Open Terminal</strong>.</p>
<p><img src="images/06_open_terminal.png%0A%20:align:%20center%0A%20:alt:%20CI/CD%20Infrastructure%20App%20Open%20Terminal" /></p></li>
<li><p>In the Web SSH Terminal that just opened, run the following command to print out Jenkins' temporary administrator password.</p>
<blockquote>

</blockquote></li>
<li><p>Double click the result from the previous step's command to copy it to your clipboard.</p>
<p><embed src="images/07_temp_admin_pass.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Master%20Temporary%20Admin%20Password" /></p></li>
<li><p>Change to the <strong>Sign in [Jenkins]</strong> tab that was previously opened. In the <strong>Administrator password</strong> field, paste in the contents of the previous step, and click <strong>Continue</strong>.</p>
<p><embed src="images/08_unlock_jenkins.png%0A%20:align:%20center%0A%20:alt:%20Unlock%20Jenkins" /></p></li>
<li><p>On the next page, click the large <strong>Install suggested plugins</strong> button.</p>
<p><embed src="images/09_suggested_plugins.png%0A%20:align:%20center%0A%20:alt:%20Install%20Jenkins%20Suggested%20Plugins" /></p></li>
<li><p>Wait for the suggested plugins to install, after which you'll be re-directed to create the first admin user. Fill in the following fields, and click <strong>Save and Continue</strong>.</p>
<ul>
<li><strong>Username</strong> - admin</li>
<li><strong>Password</strong> - any password of your choice</li>
<li><strong>Confirm password</strong> - matching password</li>
<li><strong>Full name</strong> - admin</li>
<li><strong>Email address</strong> - <script type="text/javascript">
<!--
h='&#110;&#x75;&#116;&#x61;&#110;&#x69;&#120;&#46;&#x63;&#x6f;&#x6d;';a='&#64;';n='&#110;&#x6f;&#114;&#x65;&#112;&#108;&#x79;';e=n+a+h;
document.write('<a h'+'ref'+'="ma'+'ilto'+':'+e+'">'+e+'<\/'+'a'+'>');
// -->
</script><noscript>&#110;&#x6f;&#114;&#x65;&#112;&#108;&#x79;&#32;&#x61;&#116;&#32;&#110;&#x75;&#116;&#x61;&#110;&#x69;&#120;&#32;&#100;&#x6f;&#116;&#32;&#x63;&#x6f;&#x6d;</noscript></li>
</ul>
<p><embed src="images/10_create_user.png%0A%20:align:%20center%0A%20:alt:%20Create%20Jenkins%20Admin%20User" /></p></li>
<li>On the Instance Configuration page that appears, <strong>leave</strong> the Jenkins URL as <strong>default</strong>, and click <strong>Save and Finish</strong>.</li>
<li><p>Jenkins setup is now complete, but first our Jenkins instance needs to be restarted. Click <strong>Restart</strong>, and then move on to the next section.</p>
<p><embed src="images/11_restart_jenkins.png%0A%20:align:%20center%0A%20:alt:%20Restart%20Jenkins" /></p></li>
</ol>
<h2 id="accessing-the-developer-workstation">Accessing the Developer Workstation</h2>
<p>Throughout this entire lab, we'll be running a large number of commands from our developer workstation, as it has already been configured with all the necessary software packages, the correct kubeconfig file, and is pointed at our git repository that's stored in Gitea.</p>
<p>To access the developer workstation, you have two options: 1, use the web SSH client as we did for the Jenkins Service, or 2, use your laptop's terminal or PuTTY to SSH into the workstation. Either option is perfectly valid, however we recommend you stick with whatever you're most comfortable with. Since we already covered how to use the web SSH client in the previous step, we'll cover SSH'ing in from your laptop here.</p>
<ol>
<li><p>Back in our Calm application page, navigate to the <strong>Services</strong> tab, and select the <strong>Workstation</strong> Service. In the right column that appears, <strong>copy</strong> the IP address of the service by clicking the button just to the right of the IP.</p>
<p><img src="images/12_copy_workstation_ip.png%0A%20:align:%20center%0A%20:alt:%20CI/CD%20Infrastructure%20App%20Copy%20Workstation%20IP" /></p></li>
<li><p>In your laptop's terminal, run the following commands to SSH into your workstation (be sure to subsitute in your workstation IP).</p>
<blockquote>

</blockquote></li>
<li><p>Validate that our kubeconfig and git repo are set up properly by running the following commands. Your output should be similar to the image below, but with different node names and IPs.</p>
<blockquote>

</blockquote>
<p><embed src="images/13_validate_workstation.png%0A%20:align:%20center%0A%20:alt:%20Validate%20Workstation%20Configuration" /></p></li>
</ol>
<h2 id="gitea-webhook-setup">Gitea Webhook Setup</h2>
<p>Our next configuration step is to create a webhook in Gitea, which tells Gitea to inform some server (in our case Jenkins) each time there is a new commit. Many popular git servers have this functionality, including GitHub, GitLab, and Gitea.</p>
<ol>
<li>We'll access our Gitea Service in the same manner as Jenkins, by navigating to the <strong>Overview</strong> tab of our Calm application, right clicking on the <strong>Gitea</strong> link, and opening it in a new tab.</li>
<li>It is expected to receive a warning from your browser about the site's security certificate not being trusted by your computer. This is due to the use of self signed SSL certificates during setup (which is not recommended for production workloads). Select the <strong>Proceed Anyway</strong> option (exact wording may depend on your browser).</li>
<li><p>On the Gitea homepage, click the <strong>Sign In</strong> button in the upper right.</p>
<p><embed src="images/14_gitea_home.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Homepage" /></p></li>
<li><p>Sign in with the following credentials.</p>
<ul>
<li><strong>Username</strong> - gitadmin</li>
<li><strong>Password</strong> - your password specified when launching the Calm blueprint</li>
</ul>
<p><embed src="images/15_gitea_signin.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Sign%20In" /></p></li>
<li><p>On the page that appears, click the <strong>gitadmin/hello-kubernetes</strong> repository link, then <strong>Settings</strong> along the right-hand side, and finally the <strong>Webhooks</strong> tab.</p>
<p><embed src="images/16_repo_settings.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Repository%20Settings" /></p></li>
<li><p>Click the blue <strong>Add Webhook</strong> button, in the list that appears click <strong>Gitea</strong>, and then fill in the following fields.</p>
<ul>
<li><strong>Target URL</strong> - The output of the <strong>echo $JENKINS_HOOK_URL</strong> command from the previous &quot;Developer Workstation&quot; section, should be of the format <strong><a href="http://">http://</a>&lt;jenkins-ip&gt;:8080/gitea-webhook/post</strong></li>
<li><strong>HTTP Method</strong> - Leave the default of <strong>POST</strong></li>
<li><strong>POST Content Type</strong> - Leave the default of <strong>application/json</strong></li>
<li><strong>Secret</strong> - Leave it blank (Jenkins does not require a secret by default)</li>
<li><strong>Trigger On</strong> - Leave the default of <strong>Push Events</strong> (any time a user runs &quot;git push&quot; Gitea will send the webhook)</li>
<li><strong>Branch filter</strong> - Leave the default of * (this means the webhook will be triggered for <em>any</em> branch)</li>
<li><strong>Active</strong> - Leave the Active checkbox <strong>enabled</strong>.</li>
</ul>
<p><embed src="images/17_gitea_add_webhook.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Add%20Webhook" /></p></li>
<li><p>Click the green <strong>Add Webhook</strong> button. You should receive a notification that the webhook has been added.</p>
<p><embed src="images/18_gitea_webhook_added.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Webhook%20Added" /></p></li>
<li><p>To validate the webhook is operating as expected, click the <strong>pencil</strong> to the right of the webhook. Scroll all the way to the bottom of the page, and click the teal <strong>Test Delivery</strong> button. After a moment, the page should refresh, and there should be a successful test event created. If the <strong>Response</strong> has a green <strong>200</strong> code, then everything is configured properly.</p>
<p><embed src="images/19_gitea_test_webhook.png%0A%20:align:%20center%0A%20:alt:%20Gitea%20Successful%20Test%20Webhook" /></p></li>
</ol>
<h2 id="dockerhub-setup">DockerHub Setup</h2>
<p>After a GitHub commit triggers a Jenkins build, and Jenkins successfully builds our new docker image, it needs some place to store the image. In this lab, we'll be using DockerHub, however there are many free container registries available.</p>
<ol>
<li><p>First, login to <a href="https://hub.docker.com/">DockerHub</a> (or create a free account) and click the <strong>Create Repository</strong> button.</p>
<p><embed src="images/20_dockerhub_create_1.png%0A%20:align:%20center%0A%20:alt:%20DockerHub%20Create%20Repository%20Button" /></p></li>
<li><p>Name the repository <strong>hello-kubernetes</strong>, give it a description of your choice, leave all other fields as default (be sure to leave the repo as <strong>Public</strong>), and click <strong>Create</strong>.</p>
<p><embed src="images/21_dockerhub_create_2.png%0A%20:align:%20center%0A%20:alt:%20DockerHub%20Create%20Repository" /></p></li>
</ol>
<h2 id="jenkins-credentials-creation">Jenkins Credentials Creation</h2>
<p>The first step of our Jenkins Setup is to add our various credentials to Jenkins’ credential store, which gives Jenkins the ability to authenticate to other pieces of our pipeline. We’ll first add our DockerHub credentials, which allows Jenkins to push images. <strong>TODO: Validate this statement. In many environments, you would also need to add git credentials for Jenkins to be able to read the repository, however in this particular environment, our Gitea server has our git repository marked as public, so no authentication is necessary to read the repo.</strong> Lastly, we’ll add our Karbon kubeconfig file to allow Jenkins to deploy our application directly onto our Kubernetes cluster.</p>
<ol>
<li>Log in to your Jenkins server with the credentials you created earlier (you may need to refresh your browser page due to the Jenkins reboot in a previous section).</li>
<li><p>In the Jenkins UI, select <strong>Credentials</strong> along the left, and then in the <strong>Stores scoped to Jenkins</strong> section, select the <strong>global</strong> domain.</p>
<p><embed src="images/22_jenkins_global_creds.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Global%20Credentials" /></p></li>
<li><p>Click <strong>Add Credentials</strong> along the left column.</p>
<p><embed src="images/23_jenkins_add_creds.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Add%20Global%20Credentials" /></p></li>
<li><p>Fill in the following fields to add your DockerHube credentials, and click <strong>OK</strong>.</p>
<ul>
<li><strong>Kind</strong> - leave as default (<strong>Username with password</strong>)</li>
<li><strong>Scope</strong> - leave as default (<strong>Global</strong>)</li>
<li><strong>Username</strong> - your DockerHub username (<strong>not</strong> your email)</li>
<li><strong>Password</strong> - your DockerHub password</li>
<li><strong>ID</strong> - leave blank</li>
<li><strong>Description</strong> - <strong>DockerHub Credentials</strong></li>
</ul>
<p><embed src="images/24_jenkins_dockerhub_creds.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Add%20DockerHub%20Credentials" /></p></li>
<li><p>Lastly, we’ll need to add our kubeconfig file as a credential to allow Jenkins to deploy our updated application onto our Kubernetes cluster. In our Workstation CLI, run the following commands to create a Kubernetes Service Account <strong>jenkins</strong>, and then create a Role Binding which maps our Service Account the the built-in <strong>admin</strong> role (each individual command starts with a &quot;$&quot;, they should be run one at a time, and do <strong>not</strong> include the &quot;$&quot; in the command).</p>
<blockquote>

</blockquote>
<blockquote>
<p><strong>note</strong></p>
<p>We're limiting our jenkins Service Account to a single Kubernetes namespace (default).</p>
</blockquote></li>
<li><p>We'll now replace the token in our existing kubeconfig with the token of our newly generated Service Account, which we can do in one line with the following command.</p>
<blockquote>

</blockquote></li>
<li><p>Copy the long output of that command into your buffer, and head back into the Jenkins UI. Select <strong>Add Credentials</strong> again, fill in the following fields, and click <strong>OK</strong>.</p>
<ul>
<li><strong>Kind</strong> - <strong>Kubernetes configuration (kubeconfig)</strong></li>
<li><strong>Scope</strong> - leave as default (<strong>Global</strong>)</li>
<li><strong>ID</strong> - leave blank</li>
<li><strong>Description</strong> - <strong>Karbon Kubernetes Kubeconfig</strong></li>
<li><strong>Kubeconfig</strong> - select the <strong>Enter directly</strong> radio button</li>
<li><strong>Content</strong> - paste in the output from the previous step</li>
</ul>
<p><embed src="images/25_jenkins_kubconfig.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Add%20Kubeconfig%20Credential" /></p></li>
</ol>
<h2 id="jenkins-pipeline-creation">Jenkins Pipeline Creation</h2>
<p>It's now time to create our Jenkins Pipeline. The pipeline is the crux of this entire CI/CD workload: our Gitea webhook calls this pipeline, which is then responsible for building our docker container, uploading the container to DockerHub, and deploying the new container to our Karbon Kubernetes cluster.</p>
<ol>
<li><p>In the Jenkins UI, click <strong>New Item</strong> in the upper left, enter <strong>hello-kubernetes</strong> as the name, select <strong>Pipeline</strong>, and click <strong>OK</strong>.</p>
<p><embed src="images/26_jenkins_create_pipeline_1.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Create%20Pipeline%201" /></p></li>
<li><p>Under the <strong>General</strong> section, give your pipeline a description, and leave all checkboxes as <strong>unselected</strong>.</p>
<p><embed src="images/27_jenkins_create_pipeline_2.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Create%20Pipeline%202" /></p></li>
<li><p>Under the <strong>Build Triggers</strong> section, select <strong>Poll SCM</strong>, and leave the Schedule <strong>blank</strong>. Without a schedule, Jenkins will <em>only</em> run this pipeline from a Webhook, which is desired for this setup. Leave all other checkboxes as <strong>unselected</strong>.</p>
<p><embed src="images/28_jenkins_create_pipeline_3.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Create%20Pipeline%203" /></p></li>
<li>Skip the <strong>Advanced Project Options</strong> section.</li>
<li><p>Under the <strong>Pipeline</strong> section, fill in the following fields.</p>
<ul>
<li><strong>Definition</strong> - Change the dropdown to <strong>Pipeline script from SCM</strong>, which allows us to store our Jenkinsfile in the same source code repository as our application</li>
<li><strong>SCM</strong> - Change the dropdown to <strong>git</strong></li>
<li><strong>Repositories</strong>
<ul>
<li><strong>Repository URL</strong> - Fill in your Gitea repository URL, which can be found by running <strong>echo $GIT_REPO_URL</strong> from your Workstation, and should be of the format <strong><a href="https://">https://</a>&lt;gitea-ip&gt;:3000/gitadmin/hello-kubernetes</strong></li>
<li><strong>Credentials</strong> - Leave as default <strong>none</strong> (if your git repository is private, you would need to specify your git credentials here)</li>
</ul></li>
<li><strong>Branches to build</strong> - Leave all as default</li>
<li><strong>Repository browser</strong> - Leave as default of <strong>Auto</strong></li>
<li><strong>Additional Behaviours</strong> - Leave default of none</li>
<li><strong>Script Path</strong> - Leave as default of <strong>Jenkinsfile</strong></li>
<li><strong>Lightweight checkout</strong> - Leave as default <strong>checked</strong></li>
</ul>
<p><embed src="images/29_jenkins_create_pipeline_4.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Create%20Pipeline%204" /></p></li>
<li>Click <strong>Save</strong> to save the pipeline configuration.</li>
</ol>
<h2 id="jenkins-pipeline-snippet-generator">Jenkins Pipeline Snippet Generator</h2>
<p>We'll now use the Jenkins Pipeline Syntax Snippet Generator to assist us when we go to create our Jenkinsfile in the upcoming section. Since the result of this section is a text string which will be included in our Jenkinsfile (and will be provided in the next section), it’s not required to perform the same steps on your system. However, it is good practice as it’s something you’ll likely need to do if you expand upon this example.</p>
<ol>
<li><p>Within your pipeline homepage, click the <strong>Pipeline Syntax</strong> button in the left column, and fill out the following fields.</p>
<ul>
<li><strong>Sample Step</strong> - change the dropdown to <strong>kubernetesDeploy: Deploy to Kubernetes</strong></li>
<li><strong>Kubeconfig</strong> - select the Kubeconfig that was added in a previous section</li>
<li><strong>Config Files</strong> - enter <strong>hello-kubernetes-dep.yaml</strong> (we have not created this file yet, but will in an upcoming section)</li>
<li>Leave all other options as <strong>defaults</strong></li>
</ul>
<p><embed src="images/30_jenkins_gen_pipeline.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Generate%20Pipeline%20Script" /></p></li>
<li><p>Click <strong>Generate Pipeline Script</strong>. In the text box that appears, you should see a string like this, however your <strong>kubeconfigId</strong> <em>will be different</em>. When this string is placed in a Jenkinsfile, it instructs Jenkins to deploy a certain configuration (hello-kubernetes-dep.yaml) against a particular Kubernetes cluster (in our case the cluster config is stored in the kubeconfig credential we created in an earlier section).</p>
<blockquote>

</blockquote>
<blockquote>
<p><strong>note</strong></p>
<p>The serverUrl field does not need an actual URL as that information is stored in our Kubeconfig.</p>
</blockquote></li>
<li>Optionally copy this script for later use.</li>
</ol>
<h2 id="jenkinsfile-and-yaml-creation">Jenkinsfile and Yaml Creation</h2>
<p>We'll now create our Jenkinsfile, which is the script Jenkins uses to run our Pipeline, and our Kubernetes YAML, which is what defines our application. We'll first grab some information from our Jenkins and DockerHub UIs, and then head over into our workstation to create our files.</p>
<ol>
<li>In the Jenkins UI, click the <strong>Jenkins</strong> icon in the upper left to navigate home, and then select <strong>Credentials</strong> along the left column.</li>
<li><p>Take note of the <strong>ID</strong> column in the Credentials table. These values will be unique on every system, and your specific values are needed when we create our Jenkinsfile.</p>
<p><embed src="images/31_jenkins_cred_ids.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Credentials%20IDs" /></p></li>
<li><p>In your DockerHub UI, select your <strong>hello-kubernetes</strong> repository, and along the right side, take note of the <strong>docker push &lt;your-username&gt;/hello-kubernetes:tagname</strong> field. Your username will be needed in the next step when we create our Jenkinsfile.</p>
<p><embed src="images/32_dockerhub_username.png%0A%20:align:%20center%0A%20:alt:%20DockerHub%20Username" /></p></li>
<li><p>Head over into your Workstation SSH session, and run the following commands to create our <strong>Jenkinsfile</strong>, substituting your unique credential IDs in the second and third commands, and DockerHub username in the fourth (each individual command starts with a &quot;$&quot;, they should be run one at a time, and do <strong>not</strong> include the &quot;$&quot; in the command).</p>
<blockquote>

</blockquote>
<p><embed src="images/33_create_jenkinsfile.png%0A%20:align:%20center%0A%20:alt:%20Create%20Jenkinsfile" /></p></li>
<li><p>We'll now create our two Yaml files which will define our application. The first is a <a href="https://kubernetes.io/docs/concepts/services-networking/service/">Service</a> to expose the application outside of the Karbon Kubernetes cluster, and the second is a <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/">Deployment</a> which defines the application containers. We’ll create both files within the <strong>hello-kubernetes/</strong> directory, but we’ll <em>only</em> apply the service yaml, as Jenkins will apply the deployment yaml (each individual command starts with a &quot;$&quot;, they should be run one at a time, and do <strong>not</strong> include the &quot;$&quot; in the command).</p>
<blockquote>

</blockquote>
<p><embed src="images/34_create_yaml.png%0A%20:align:%20center%0A%20:alt:%20Create%20Application%20YAML" /></p>
<blockquote>
<p><strong>note</strong></p>
<p>Take note of the ${GIT_COMMIT} value in the deployment YAML. Jenkins will automatically substitute in the git commit ID, so each time the deployment is applied, the image tag is incremented, and the pods are re-deployed.</p>
</blockquote></li>
<li><p>Now that our Service is deployed, and our local files are written, it’s time to commit and push changes to our repository with the following commands.</p>
<blockquote>

</blockquote>
<p><embed src="images/35_git_add_files.png%0A%20:align:%20center%0A%20:alt:%20Git%20Add%20and%20Commit%20Jenkinsfile%20and%20App%20YAML%20Files" /></p></li>
</ol>
<h2 id="manual-build-and-application-deployment">Manual Build and Application Deployment</h2>
<p>Typically, running <strong>git push</strong> will trigger a Jenkins build through the GitHub webhook, however this will not work until we manually trigger a build. This is because the SCM details (including the project URL) in the Jenkins pipeline are not initialized until the first build, and without those details Jenkins is not able to determine the correlation between the webhook and the pipeline. Let’s manually kick off a build to get things started.</p>
<ol>
<li><p>In the Jenkins UI, navigate to our <strong>hello-kubernetes</strong> Pipeline, and click the <strong>Build Now</strong> link in the left column.</p>
<p><embed src="images/36_jenkins_manual_build.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Manual%20Build" /></p></li>
<li><p>Build #1 should appear in the <strong>Build History</strong> section in the left column. Click the <strong>#1</strong> link, and then select <strong>Console Output</strong> in the left column. This allows us to monitor the status of the Jenkins build. At the top of the build, we should see a successful login to DockerHub.</p>
<p><embed src="images/37_jenkins_console_1.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Build%20#1%20Console%20Output%201" /></p></li>
<li><p>In the middle of our console output we should see the docker image being successfully built.</p>
<p><embed src="images/38_jenkins_console_2.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Build%20#1%20Console%20Output%202" /></p></li>
<li><p>At the bottom of the console output we should see our image being tagged, pushed to DockerHub, and then finally our <strong>kubernetesDeploy</strong> task deploying our containers to our Karbon Kubernetes cluster.</p>
<p><embed src="images/39_jenkins_console_3.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Build%20#1%20Console%20Output%203" /></p></li>
<li><p>In DockerHub, we can validate that our newly pushed container is present, with both our GIT_COMMIT and <strong>latest</strong> labels.</p>
<p><embed src="images/40_dockerhub_tags.png%0A%20:align:%20center%0A%20:alt:%20DockerHub%20Repository%20Tags" /></p></li>
<li><p>We can also validate through the command line that our pods have been deployed, and our application Service has an IP by running the following commands from our Workstation.</p>
<blockquote>

</blockquote>
<p><img src="images/41_kubectl.png%0A%20:align:%20center%0A%20:alt:%20Kubectl%20Get%20Pods%20/%20Svc" /></p></li>
<li><p>We can then access our application via the <strong>External-IP</strong> value of the hello-kubernetes service (10.45.100.46 in my case). Be sure to refresh the page several times to see the pod change.</p>
<p><embed src="images/42_hello_nutanix.png%0A%20:align:%20center%0A%20:alt:%20Hello%20Nutanix%20Application" /></p></li>
</ol>
<h2 id="automated-application-deployment-through-a-git-push">Automated Application Deployment Through a Git Push</h2>
<p>If you’ve made it this far, congratulations! We’re finally at a point where we can kick off fully automated builds and deployments. To do so, we need to commit and push a change in our application code. So our change is obvious, we’ll change the <strong>Hello Nutanix!</strong> message to <strong>Hello CI/CD!</strong>.</p>
<ol>
<li><p>From within the <strong>hello-kubernetes/</strong> directory on your workstation, run the following commands to change the code, add the change, commit the change, and finally push the change.</p>
<blockquote>

</blockquote>
<p><embed src="images/43_git_push_new_code.png%0A%20:align:%20center%0A%20:alt:%20Git%20Push%20New%20Application%20Code" /></p></li>
<li><p>As soon as you run git push, you should see an automated build started in your Jenkins project.</p>
<p><embed src="images/44_jenkins_build_2.png%0A%20:align:%20center%0A%20:alt:%20Jenkins%20Automatic%20Build%202" /></p></li>
<li><p>Once the build is complete, let’s first verify we have new pods deployed via the command line.</p>
<blockquote>

</blockquote>
<p><embed src="images/45_kubectl-get-pods.png%0A%20:align:%20center%0A%20:alt:%20Kubectl%20Get%20Pods%20after%20Git%20Push" /></p></li>
<li><p>Finally, refresh our application page to view the updated message.</p>
<p><img src="images/46_hello_cicd.png%0A%20:align:%20center%0A%20:alt:%20Hello%20CI/CD%20Application" /></p></li>
</ol>
<h2 id="takeaways">Takeaways</h2>
<p>While setting up a CI/CD pipeline can be quite a bit of effort, the value it brings to your organization makes it well worth it. Once configured, a simple git push -- an operation your developers likely run several times a day -- results in a brand new application, with minimal to no effort on your or the developers part. This can be further expanded into advanced techniques like Canary releases or A/B testing. Thanks for reading!</p>
<h2 id="optional-use-nutanix-calm-jenkins-plugin">(Optional) Use Nutanix Calm Jenkins Plugin</h2>
<p>In this lab, we utilized the <a href="https://wiki.jenkins.io/display/JENKINS/Kubernetes+Continuous+Deploy+Plugin">KubernetesCContinuousDDeploy</a> Jenkins Plugin to deploy our new docker containers. Another option would be to utilize the <a href="https://wiki.jenkins.io/display/JENKINS/Nutanix+Calm+Plugin">NutanixCCalm</a> Jenkins Plugin to call a Calm application action to deploy our new docker containers. Can you change the existing Jenkins Pipeline to utilize the Nutanix Calm plugin instead?</p>
<p><strong>Hints</strong></p>
<ul>
<li>You'll first want to define our <strong>hello-kubernetes</strong> application as a Calm Application Blueprint, rather than the YAML we were using.</li>
<li>Once you successfully build the blueprint, create a <strong>profile action</strong> which accepts a runtime variable (which represents the docker label / tag) and makes an API call into the Kubernetes API to update the containers.</li>
<li>Once that is built, utilize the Jenkins syntax generator to create the relevant Jenkinsfile snippet to call your Calm Application Action.</li>
<li>Substitute out the Kubernetes Deploy snippet with the Calm Application Action snippet in your Jenkinsfile.</li>
</ul>
</html>