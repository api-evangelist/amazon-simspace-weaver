---
title: "Accelerating HPC Deployment with AWS Parallel Computing Service and Kiro CLI"
url: "https://aws.amazon.com/blogs/hpc/accelerating-hpc-deployment-with-aws-parallel-computing-service-and-kiro-cli/"
date: "Thu, 26 Mar 2026 18:06:41 +0000"
author: "Markus Adhiwiyogo"
feed_url: "https://aws.amazon.com/blogs/hpc/feed/"
---
<p>Research teams moving from on-premises HPC environments often struggle with the complexity of cloud deployment. Traditional approaches require deep expertise in AWS networking, storage architectures, and Slurm configuration management. A typical manual deployment involves weeks of infrastructure provisioning, network topology design, scheduler configuration, and performance tuning. Research teams with limited platform engineering resources find themselves caught between the urgent need for computational capacity and the technical barriers to deployment.</p> 
<p>In this post, we’ll show you how to use the Kiro CLI to automatically deploy and configure AWS PCS clusters with industry best practices built in. We’ll walk through creating Custom Agents that handle infrastructure provisioning, monitoring setup, and cost optimization while maintaining the flexibility research teams need for their specific workloads.</p> 
<h2>HPC in the Cloud</h2> 
<p>Cloud-based HPC has become standard practice due to its flexible compute access, broad resource availability, and valuable infrastructure abstraction. In this post, we’ll utilize AWS Parallel Computing Service (PCS) and Kiro CLI to demonstrate this approach.</p> 
<p><strong>AWS PCS</strong>: AWS PCS simplifies HPC deployment by providing a managed Slurm scheduler with automated configuration, scaling, and maintenance. The service handles cluster lifecycle management while allowing customers to maintain control over compute resources and job scheduling policies. PCS integrates natively with AWS services like Elastic File System for shared storage and CloudWatch for monitoring, reducing the integration work typically required for HPC deployments.</p> 
<p><strong>Kiro CLI</strong>: Kiro CLI becomes exceptionally powerful when you define <a href="https://kiro.dev/docs/cli/custom-agents/" rel="noopener noreferrer" target="_blank">custom agents</a> with specialized guidance tailored to your specific use cases. By encoding domain expertise directly into agent definitions, you can create purpose-built automation that understands your organization’s best practices and requirements. For example, we’ll create a PCS deployment agent that incorporates AWS Parallel Computing Service best practices for HPC workloads—including optimal instance selection, network configuration, and storage optimization. This agent will leverage established deployment patterns to streamline complex cluster provisioning, reducing both deployment time and the risk of configuration errors. Rather than generic automation, this approach delivers intelligent, context-aware assistance that adapts proven methodologies to your specific infrastructure needs.</p> 
<h2>Getting Started</h2> 
<p>Before implementing the automated PCS deployment approach, you need three prerequisites:</p> 
<ul> 
 <li>Install the Kiro CLI following the <a href="https://kiro.dev/docs/cli/" rel="noopener noreferrer" target="_blank">installation guide</a></li> 
 <li>Download the demonstration code from the <a href="https://github.com/aws-samples/sample-pcs-kiro-agent" rel="noopener noreferrer" target="_blank">pcs-kiro demo repository</a></li> 
 <li>Ensure you have local AWS credentials, you can check by running <code>aws sts get-caller-identity</code></li> 
 <li>(Optional) Configure the <a href="https://aws.amazon.com/about-aws/whats-new/2025/07/aws-knowledge-mcp-server-available-preview/" rel="noopener noreferrer" target="_blank">AWS Knowledge MCP Server</a> using the <a href="https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/qdev-mcp.html" rel="noopener noreferrer" target="_blank">MCP configuration guide</a></li> 
</ul> 
<h2>Using the Agent</h2> 
<p>First we will initialize the agent.We’ll want to enable the todo list setting on Kiro. This enables more in depth planning for the agent, as it goes through creating a cluster it will create list of tasks and go step by step:</p> 
<p><code>kiro settings chat.enableTodoList true</code></p> 
<p>Start the interactive PCS deployment agent:</p> 
<p><code>kiro chat --agent pcs-interactive-agent</code></p> 
<h4>Discovery phase</h4> 
<p>Now we will ask the agent to help us create a cluster.</p> 
<p><code>User: I am new to PCS. Help me create my first cluster with best practices.</code></p> 
<p>The agent will first check for active AWS credentials, then respond with some discovery questions if it isn’t sure about details, it will also explore your existing environment, and provide recommendations based on best practices. This is helpful for situations where you want the agent to use existing VPCs or other AWS resources.</p> 
<p><img alt="Agent &quot;Resource Scan Complete&quot; output" class="aligncenter wp-image-5637 size-full" height="467" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-1-1.png" width="904" /></p> 
<p><img alt="Agent &quot;Smart Recommendations&quot; output" class="aligncenter wp-image-5619 size-large" height="410" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-2-1024x410.png" width="1024" /></p> 
<p><img alt="Agent &quot;Questions to Get Started&quot; output" class="aligncenter wp-image-5620 size-large" height="317" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-3-1024x317.png" width="1024" /></p> 
<p>Here you can tweak or provide details and the agent will do a final summary and begin creating infrastructure in your AWS account.</p> 
<p><img alt="Example natural language input prompt to agent" class="aligncenter wp-image-5623 size-large" height="313" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-6-1024x313.png" width="1024" /></p> 
<h3>Infrastructure Creation</h3> 
<p>The provided is a snippet of the infrastructure creation process, depending on your environment and your requirements this output may change.</p> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">Agent:

VPC Created:vpc-█████████████████ (10.3.0.0/16)  

- Public Subnet:subnet-████████████ (10.3.0.0/20) - for login nodes
- Private Subnet:subnet-████████████ (10.3.32.0/20) - for compute nodes
- Internet Gateway:igw-████████████
- NAT Gateway:nat-████████████ (52.38.92.80)
- Route Tables:Public (rtb-████████████), Private (rtb-01484185571aabba1)

Security Groups Created: 

- Cluster SG:sg-████████████ (Slurm ports 6817-6818)
- SSH SG:sg-████████████ (SSH port 22)

IAM Role Created:AWSPCS-scientific-cluster-role  

- Instance Profile:AWSPCS-scientific-cluster
- Policies:AmazonSSMManagedInstanceCore + PCS registration

Cluster Created:pcs_xfzpdypb7z (scientific-cluster)  

- Status:CREATING → Getting endpoints
- Size:SMALL (up to 100 nodes, 1000 jobs)
- Slurm Version:24.11
- Controller Endpoint:10.3.45.99:6817
  
Launch Templates Created: 

- Compute:lt-████████████ (c6i.xlarge)
- Login:lt-████████████ (c6i.xlarge + SSH key)

Compute Node Groups Created: 

- Login Group:pcs_ndgfaikhwl (1 c6i.xlarge in public subnet)
- Compute Group:pcs_████████████ (0-10 c6i.xlarge in private subnet)
- AMI Used:ami-████████████ (Slurm 24.11)
- Status:CREATING (takes 3-5 minutes)
  
Queue Created:pcs_bogst8bszr (scientific)  

- Status:CREATING → ACTIVE
- Targets:Compute node group (pcs_dficwpzin2)
- Purpose:Scientific computing workloads</code></pre> 
</div> 
<p>The agent will then provide details after infrastructure creation is complete. There may be times where a deployment of an AWS resource fails. The instructions the agent has been provided will tell it to troubleshoot those issues and fix them before continuing. The Agent also creates a cluster summary and documentation in the /generated directory that shows exactly what was built and how it was configured.</p> 
<p><img alt="Example agent output when First PCS Cluster is ready" class="aligncenter wp-image-5624 size-full" height="286" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-7.png" width="917" /><br /> Once complete, the agent will show you how to connect to and test your cluster and provide cost estimates</p> 
<p><img alt="Agent's direction to connect and test your cluster" class="aligncenter wp-image-5625 size-full" height="1101" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-8.png" width="986" /></p> 
<p>In this small interaction we’ve created a reasonably complex architecture, with a fully functioning network, controller, queue, compute and login nodes, and appropriate templates and roles. All this just by defining our general needs and use case.</p> 
<p><img alt="Example cluster architecture generated by agent" class="aligncenter wp-image-5626 size-full" height="900" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/03/26/HPCBlog-404-9.png" width="981" /></p> 
<p>Figure 1: Architecture diagram showing AWS Parallel Computing Service (PCS) cluster deployment within a Virtual Private Cloud (VPC).</p> 
<p>The diagram displays a PCS Controller connected to a PCS Queue, both positioned outside the VPC. Inside the VPC are two subnets: a public subnet containing a Login Node Group with one c6i.xlarge instance using a static launch template, and a private subnet containing a Compute Node Group with c6i.xlarge instances (0-10 capacity) using a dynamic launch template. Both node groups include role assignments and are represented with AWS service icons including compute instances and launch template symbols.</p> 
<p>All together this processes took about 30 minutes with some user back and forth, and we can immediately log in and start running jobs. This architecture can be extended to include things like shared storage and additional compute node groups and queues.</p> 
<h2>Suggested Best Practices</h2> 
<ul> 
 <li>Be direct, verbose and prescriptive with agent instructions 
  <ul> 
   <li>As an example: In our architecture we didn’t ask for a filesystem but we easily could have asked for an FSx Lustre distributed file system to be set up as well</li> 
  </ul> </li> 
 <li>Monitor deployments and cancel commands if confused</li> 
 <li>Allow occasional failures – agents tend to recovery</li> 
 <li>Customize agent configurations for your specific use case</li> 
 <li>Provide comprehensive workload context and requirements</li> 
</ul> 
<h2>Known Limitations</h2> 
<ul> 
 <li>Agents may lose track of details – remind them as needed</li> 
 <li>Manual verification required – check results match your requirements</li> 
</ul> 
<h2>Conclusion</h2> 
<p>AWS Parallel Computing Service (PCS) provides research teams with a fully managed HPC service that eliminates the operational complexity of running Slurm clusters at scale. Combined with Kiro CLI’s multi-agent architecture, teams can now deploy production-ready HPC environments in hours rather than weeks. This approach is particularly valuable for research organizations that need computational power without the overhead of managing underlying infrastructure.</p> 
<p>Happy Building!</p>
