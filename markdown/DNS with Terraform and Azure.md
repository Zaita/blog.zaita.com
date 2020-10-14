<h1 id="azure-dns-with-terraform">Azure DNS with Terraform</h1>
<h2 id="overview">Overview</h2>
<p>I work as a Solutions, Security and Software Architect. Due to this combination, I am often in conversations around deployment of modern infrastructure using zero trust principles. One recent conversation was around the management of Azure DNS records using Terraform.</p>
<p>Terraform is a declarative language tool for the provisioning of cloud assets. It’ll create the assets and apply some basic configuration, but it is not a configuration tool. One of the key issues we had is that Terraform is strict on managing an entire object. That is, you cannot get Terraform to manage only a small component of an object… like an individual DNS record within an existing DNS Zone.</p>
<p>In my personal projects, I try to mimic zero trust and enterprise thinking. This was a good opportunity to look at how I was managing the DNS for my own domains (<a href="http://zaita.co">zaita.co</a>, <a href="http://zaita.io">zaita.io</a> and <a href="http://zaita.com">zaita.com</a>) for the different projects I have hooked up. How could I have a nice, tidy and automated DNS management system that used Terraform, but didn’t require me to merge all projects into a single monolithic project.</p>
<h2 id="dns-subdomain-delegation">DNS Subdomain Delegation</h2>
<p>One of the first ideas that I came up with was to delegate subdomains to new DNS Zones. That would mean a domain like <a href="http://blog.zaita.com">blog.zaita.com</a> would be able to have it’s own Azure DNS Zone configuration and add it’s own subdomains (e.g. <a href="http://staging.blog.zaita.com">staging.blog.zaita.com</a>) without engaging with the top level DNS Zone.</p>
<p>The pros of this are:</p>
<ul>
<li>I can manage my top level DNS in a single zone file</li>
<li>Individual projects get full control over their own DNS records</li>
<li>Terraform compatible solution</li>
</ul>
<p>The cons of this are:</p>
<ul>
<li>Management of the top level DNS zone is still manual</li>
<li>Individual projects get a lot of control over their subdomain and can delegate further</li>
<li>There is no visibility on changes being made to subdomain records</li>
</ul>
<p>While I think this is likely a viable approach for many businesses, the con of having no visibility of subdomain changes through a central mechanism was the tipping point for me. I would like to see my domain records in a single location.</p>
<h2 id="azure-cli-configuration">Azure CLI Configuration</h2>
<p>The next idea that was floated was to allow projects to use Azure CLI to edit the master DNS Zone file. Service Principal accounts for each project would be given permission to make changes.</p>
<p>The pros of this are:</p>
<ul>
<li>No changes to existing projects/configurations needs to be made</li>
<li>It’s really quick and easy to accomplish, most Terraform deployments use some form of Azure CLI already</li>
</ul>
<p>The cons of this are:</p>
<ul>
<li>Every service principal needs to have access to modify the Azure DNS Zone file</li>
<li>We’re unable to easily track changes and the origins of changes to the DNS Zone file</li>
<li>Too easy for someone to accidentally break the configuration of another project</li>
</ul>
<p>Ultimately, I think the security concerns around giving broad access to all service principals makes this a non-viable approach. This goes pretty strongly against a least privileges security model.</p>
<h2 id="git-repository-with-azure-devops">Git Repository with Azure DevOps</h2>
<p>The solution I settled on, using a Git repository to store the Terraform configurations for the master DNS Zone, and allowing projects to make pull requests with the changes they want.</p>
<p>Individual projects can automate the pull requests to the Git repository, a manual approval process is required for a sanity check of all changes proposed, and merging et al can be automated without any manual steps.</p>
<p>This approach I found to be the best for me. The pros of this approach are:</p>
<ul>
<li>I can use Terraform to manage the master DNS Zone</li>
<li>I do not need to allow DNS delegation for all subdomains</li>
<li>Tracking changes is native functionality in Git</li>
<li>Deployment of changes can be automated using Azure DevOps</li>
<li>There is a manual approval (1 button push) step for sanity checking</li>
<li>I don’t have to change existing service principal permissions</li>
</ul>
<p>The cons for this approach:</p>
<ul>
<li>Projects do need to create a pull request for the Git repository. This functionality isn’t natively supported in Terraform, so some bash script would be required.</li>
</ul>

