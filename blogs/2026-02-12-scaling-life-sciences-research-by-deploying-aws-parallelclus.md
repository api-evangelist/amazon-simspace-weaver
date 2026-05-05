---
title: "Scaling life sciences research by deploying AWS ParallelCluster and AWS DataSync"
url: "https://aws.amazon.com/blogs/hpc/scaling-life-sciences-research-by-deploying-aws-parallelcluster-and-aws-datasync/"
date: "Thu, 12 Feb 2026 06:16:23 +0000"
author: "Yoshimasa Aoto"
feed_url: "https://aws.amazon.com/blogs/hpc/feed/"
---
<p>In life sciences research, managing large-scale computational resources and data efficiently is important for success. However, traditional on-premises environments often struggle to meet these requirements effectively. This post demonstrates how JSR Corporation transformed their research infrastructure using <a href="https://aws.amazon.com/hpc/parallelcluster/">AWS ParallelCluster</a> and <a href="https://aws.amazon.com/datasync/">AWS DataSync</a>, achieving a 33% reduction in CPU usage and 85% in storage requirements. JSR’s research teams now spend more time on genomics and medical research. In this blog post, Yoshimasa Aoto from the JSR-Keio University Medical and Chemical Innovation Center (JKiC) shares their challenges and the benefits of adopting AWS.</p> 
<h2>JSR’s life sciences business initiatives</h2> 
<p>JSR’s academic-industrial research facility aims to expand drug discovery and development support services through medical and biological research using novel modalities. Their bioinformatics and medical informatics research require significant computational resources, which they previously managed through an on-premises data center.</p> 
<h2>Challenges with on-premises high performance computing</h2> 
<p>As JSR’s on-premises servers aged and approached their refresh cycle, several challenges emerged that prompted the company to seriously consider AWS adoption:</p> 
<p><strong>Bottlenecks from Sporadic Large-Scale Computations</strong>: Daily analytical workloads, such as genomic sequence alignment, protein structure prediction, and medical image analysis, typically consumed only 25-50% of the existing server’s CPU resources. However, each research project generated additional sporadic large-scale computations every few weeks to months. These experiments produced 10GB to 15TB of data per batch, creating consistent bottlenecks in CPU, memory, or storage resources.</p> 
<p><strong>Growing Data Management Complexity</strong>: Life sciences research facilities commonly face challenges managing large experimental datasets. JSR’s laboratory deals with many precious data samples that are difficult to recollect, and since some experiments cost up to $50,000, the data needs to be stored safely for long periods of time. Previously, researchers manually uploaded data from experimental equipment to servers and created separate physical storage backups, a process that became increasingly cumbersome as research data volume grew.</p> 
<p><strong>GPU Procurement Challenges</strong>: JSR also struggled with GPU resources for machine learning workloads. Traditional hardware procurement cycles required annual budgeting and planning, making it difficult to keep pace with rapid AI technology advances. While they used <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2)</a> on-demand instances as a stopgap solution, managing the startup/shutdown cycles and associated costs created psychological barriers for users.</p> 
<h2>Implementing a hybrid cloud HPC architecture</h2> 
<p>To address these challenges, JSR implemented AWS services, creating a hybrid cloud system that balanced flexibility with control.</p> 
<p><strong>Flexible compute with AWS ParallelCluster</strong></p> 
<p>AWS ParallelCluster is an open-source high-performance computing (HPC) tool that deploys and manages HPC clusters on AWS. Customers often use it for flexible resource management of sporadic computations and high-volume storage requirements. By customizing <a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/configuration-of-multiple-queues-v3.html">Slurm partitions (queues)</a> in AWS ParallelCluster, JSR achieved an architecture where appropriate instance types and sizes automatically start based on computational demands and stop after job completion. They implemented both CPU and GPU-oriented partitions, creating a flexible and cost-optimized HPC configuration that easily adapts to changing requirements. JSR favors the <a href="https://aws.amazon.com/ec2/instance-types/g6e/">G6e instance family</a> for its excellent price-performance balance and is actively adopting it for computational workloads. They are also currently evaluating running their <a href="https://www.nextflow.io/">Nextflow</a> genomics analysis pipeline on <a href="https://aws.amazon.com/batch/">AWS Batch</a>.</p> 
<p><strong>Automated data management with AWS DataSync</strong></p> 
<p>Additionally, JSR implemented automated data transfer and backup using AWS DataSync. AWS DataSync is an online data movement service that simplifies and accelerates data migrations to and from AWS. It automatically transfers experimental data to both analysis environments and backup storage. The solution leverages AWS DataSync’s built-in scheduling capabilities for daily transfers during off-peak hours, automatic retry mechanism for failed transfers, and task-level monitoring with <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch</a> for ensuring data transfer reliability.</p> 
<p>Furthermore, JSR optimized costs by leveraging different <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3)</a> storage classes – using <a href="https://aws.amazon.com/s3/storage-classes/glacier/">Amazon S3 Glacier</a> Deep Archive for backup data and <a href="https://aws.amazon.com/s3/storage-classes/intelligent-tiering/">Amazon S3 Intelligent-Tiering</a> for analysis data.<br /> <strong>Complete integration with a hybrid HPC architecture</strong></p> 
<p>Figure 1 illustrates JSR’s hybrid architecture and shows how experimental data flows from on-premises resources through automated DataSync transfers to AWS storage, while compute workloads dynamically scale using AWS ParallelCluster.</p> 
<div class="wp-caption aligncenter" id="attachment_5585" style="width: 1930px;">
 <img alt="" class="size-full wp-image-5585" height="1440" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/02/10/image001-2.png" width="1920" />
 <p class="wp-caption-text" id="caption-attachment-5585">Figure 1: JSR’s Large-Scale Computing Environment for Life Sciences Research</p>
</div> 
<p>The architecture supports the following workflow:</p> 
<ol> 
 <li>Researchers generate experimental data on-premises</li> 
 <li>DataSync automatically transfers data to S3 during off-peak hours</li> 
 <li>AWS ParallelCluster provisions compute resources based on job requirements</li> 
 <li>Analysis results are stored in S3 Intelligent-Tiering</li> 
 <li>Backup data moves to Glacier Deep Archive for long-term retention</li> 
</ol> 
<h2>Benefits</h2> 
<p>The implementation of AWS managed services delivered measurable benefits to JSR’s life sciences research:</p> 
<p><strong>Downsized On-Premises Infrastructure</strong>: By offloading sporadic large-scale computations to AWS, JSR eliminated the need for excess on-premises resources, reducing their server CPU count by 33% and storage capacity by 85%.</p> 
<p><strong>Automated CPU and GPU Resource Management</strong>: GPU resource procurement, which previously took about one year, now takes just minutes, eliminating research bottlenecks. Managed services also simplified cost management.</p> 
<p><strong>Reduced Infrastructure Deployment Risk</strong>: The solution eliminated approximately six months of administrative overhead related to cost estimation, budget approval, procurement, installation, and configuration. It also removed the risk of stranded assets due to changing requirements.</p> 
<p><strong>Automated Data Management</strong>: Previously, data collection from measurement devices and transfer to analysis servers and backup storage could delay research by up to a week. Automation has eliminated these delays, significantly improving research efficiency.</p> 
<h2>Conclusion</h2> 
<p>JSR’s journey demonstrates how life sciences organizations can transform their research capabilities by modernizing their HPC infrastructure with AWS. Their success shows that the right cloud strategy can not only reduce operational overhead but also accelerate scientific discovery. Before AWS adoption, JSR faced these key challenges with their on-premises data center HPC environment:</p> 
<ul> 
 <li>Bottlenecks from sporadic large-scale computations</li> 
 <li>Growing data management complexity</li> 
 <li>GPU procurement difficulties</li> 
</ul> 
<p>The implementation of AWS ParallelCluster and AWS DataSync delivered substantial improvements: on-premises infrastructure needs dropped by 33% for CPU and 85% for storage; GPU procurement that once took a year now happens in minutes; six months of planning overhead disappeared; and researchers gained an additional week of productive time per experiment that was previously lost to data management tasks. To learn more about implementing similar solutions, see the&nbsp;<a href="https://docs.aws.amazon.com/parallelcluster/latest/ug/what-is-aws-parallelcluster.html">AWS ParallelCluster documentation</a>&nbsp;and&nbsp;<a href="https://docs.aws.amazon.com/datasync/latest/userguide/setting-up.html">AWS DataSync Getting Started Guide</a>.</p> 
<p><strong>About JSR Corporation</strong></p> 
<p><img alt="" class="size-full wp-image-5586 alignleft" height="202" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/02/10/image002-1.png" width="202" /></p> 
<p><em>JSR Corporation operates under the corporate philosophy of “Materials Innovation – Creating Value through Materials, Contributing to Society (People, Society, and the Environment).” The company strives to provide irreplaceable materials to society while earning public trust through its contributions. In the life sciences sector, JSR delivers CDMO/CRO services for drug discovery support.</em></p>
