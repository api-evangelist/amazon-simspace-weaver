---
title: "Optimize Nextflow Workflows on AWS Batch with Mountpoint for Amazon S3"
url: "https://aws.amazon.com/blogs/hpc/optimize-nextflow-workflows-on-aws-batch-with-mountpoint-for-amazon-s3/"
date: "Tue, 13 Jan 2026 15:19:37 +0000"
author: "Michael Mueller"
feed_url: "https://aws.amazon.com/blogs/hpc/feed/"
---
<p><em>This post was contributed by Michael Mueller (AWS) and Simon Murray (Genomics England)</em></p> 
<p>Are you running genomic workflows with <a href="https://www.nextflow.io/">Nextflow</a> on <a href="https://aws.amazon.com/batch/">AWS Batch</a> and experiencing bottlenecks when staging large reference files? In this post, we will show you how to optimize your workflow performance by leveraging <a href="https://aws.amazon.com/s3/features/mountpoint/">Mountpoint for Amazon S3</a> to stream reference data directly into your Nextflow processes, eliminating the need to stage large static files repeatedly.</p> 
<h2>Background</h2> 
<p>When running Nextflow workflows on AWS Batch, input files typically need to be staged from <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3)</a> to <a href="http://aws.amazon.com/ebs/">Amazon Elastic Block Store (Amazon EBS)</a>. For large reference files, this staging process can create significant bottlenecks, especially when the same files are copied repeatedly for different process executions.</p> 
<h2>Mountpoint for Amazon S3: a better approach</h2> 
<p>Mountpoint for Amazon S3 is a file client that provides a POSIX-like interface to Amazon S3 object storage. It’s specifically optimized for read access to large objects, making it ideal for genomic workflows that require access to large reference files. With Mountpoint, you can mount an S3 bucket as a local file system and stream data directly into your processes without copying the entire file to local storage first.</p> 
<h2>Understanding Nextflow data handling</h2> 
<p>In Nextflow, there are two ways to reference input files:</p> 
<ol> 
 <li><em>Dataflow Channels:</em> The recommended approach for files that are passed between processes and represent intermediate results that can create dependencies between workflow tasks that Nextflow manages when orchestrating the workflow execution.</li> 
 <li><em>Direct Path References:</em> Suitable for static reference files that do not change between runs and are not passed between processes and therefore do not impact orchestration of workflow tasks.</li> 
</ol> 
<h2>How Genomics England leverages Mountpoint for Amazon S3 to accelerate their variant annotation pipeline</h2> 
<p>Genomics England employs a Nextflow workflow that runs the <a href="https://www.ensembl.org/info/docs/tools/vep/index.html">Variant Effect Predictor tool</a>, which requires over 500 GB of reference data. Initially, the team considered using Nextflow’s built in S3 integration. However, this approach quickly proved impractical due to the lengthy staging times for such large datasets. To overcome this challenge, Genomics England adopted Mountpoint for Amazon S3 for the reference data, eliminating the need for time-consuming file staging. This optimization significantly accelerated their variant annotation pipeline.</p> 
<h2>Solution architecture</h2> 
<p>Figure 1 shows how reference data stored in an Amazon S3 bucket is accessed from a Nextflow process running in a Docker container using Mountpoint for Amazon S3. Other input data is staged to the Amazon EBS volume attached to the <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2)</a> instance.</p> 
<div class="wp-caption aligncenter" id="attachment_5538" style="width: 1034px;">
 <img alt="Figure 1: AWS Batch using Mountpoint for Amazon S3 architecture. Reference data is streamed into the Nextflow process container via the Mountpoint for Amazon S3 client that presents the Amazon S3 bucket as a directory on the local file system (1). &nbsp;The mount path is mapped into the Docker container as a volume (2). Other input data is staged to the Amazon EBS volume attached to the Amazon EC2 instance through Nextflow’s standard Amazon S3 integration (3)." class="size-large wp-image-5538" height="811" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/01/05/CleanShot-2026-01-05-at-10.45.32-1024x811.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5538">Figure 1: AWS Batch using Mountpoint for Amazon S3 architecture. Reference data is streamed into the Nextflow process container via the Mountpoint for Amazon S3 client that presents the Amazon S3 bucket as a directory on the local file system (1). &nbsp;The mount path is mapped into the Docker container as a volume (2). Other input data is staged to the Amazon EBS volume attached to the Amazon EC2 instance through Nextflow’s standard Amazon S3 integration (3).</p>
</div> 
<p>Here’s how to set up your environment to use Mountpoint for Amazon S3 in your Nextflow workflows.</p> 
<h3>Prerequisites</h3> 
<ul> 
 <li>An <a href="https://aws.amazon.com/account/">AWS account</a> with appropriate permissions</li> 
 <li><a href="https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html">AWS Batch environment configured</a></li> 
 <li><a href="https://www.nextflow.io/docs/latest/install.html">Nextflow (community edition) installed</a><br /> (Nextflow 24.10.4 with nf-amazon plugin version 2.9.0 was used for this blog post)</li> 
 <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/mountpoint-usage.html">Access to reference data in an S3 bucket</a></li> 
</ul> 
<h3>Walkthrough</h3> 
<h4>1.&nbsp;&nbsp;&nbsp;&nbsp; Add Mountpoint client installation to EC2 launch template</h4> 
<p>You should already have a <a href="https://docs.aws.amazon.com/batch/latest/userguide/launch-templates.html">launch template</a> associated with your AWS Batch compute environment. In the <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html"><em>User Data</em></a> section add the bash commands that will install the Mountpoint client and mount your S3 bucket:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">curl -s https://s3.amazonaws.com/mountpoint-s3-release/latest/x86_64/mount-s3.rpm -o /mount-s3.rpm
dnf -y install /mountpoint-s3.rpm
mkdir -p /my-s3-mountpoint
mount-s3 my-s3-bucket /my-s3-mountpoint
</code></pre> 
</div> 
<p>Please note Mountpoint requires the name of an S3 bucket not the S3 URI.</p> 
<p>Running Docker containers as non-root users is considered a security best practice as it reduces the potential impact of container breakouts and follows the principle of least privilege. By default, users other than the user who ran the <code>mount-s3</code> command cannot access your mounted directory, even if the permissions and ownership settings would allow it. This is true even for the root user and is a limitation of the FUSE system Mountpoint uses to create a file system.</p> 
<p>When running Docker containers as non-root users, modify your Mountpoint command in the EC2 launch template&nbsp;to allow non-root users access to the mount:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">mount-s3 my-s3-bucket /my-s3-mount-point –allow-other</code></pre> 
</div> 
<p>To use this flag, you may need to first configure FUSE by adding the line user_allow_other to the /etc/fuse.conf file. Even with this flag enabled, Mountpoint still respects the permissions and ownership <a href="https://github.com/awslabs/mountpoint-s3/blob/main/doc/CONFIGURATION.md#file-system-configuration">configured with other flags</a>.</p> 
<h4>2.&nbsp;&nbsp;&nbsp;&nbsp; Configure Nextflow volume</h4> 
<p>In the AWS Batch configuration of your Nextflow pipeline, add the Mountpoint for Amazon S3 directory as a volume:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">aws {
  batch {
      volumes = ‘/my-s3-mount-point:/my-s3-mount-point:r’
  }
}
</code></pre> 
</div> 
<h4>3.&nbsp;&nbsp;&nbsp;&nbsp; Pass reference file path to Nextflow process</h4> 
<p>To stream data directly from Amazon S3 without staging, the path to the Mountpoint for Amazon S3 directory must be passed into the Nextflow process using the val input directive. (Note that using the path input directive with a Mountpoint path will result in an error during data staging.) However, continue to pass non-static data using the path directive.</p> 
<p>Mountpoint paths passed as val can be referenced directly in the script block:</p> 
<div class="hide-language"> 
 <pre><code class="lang-ruby">process VEP {

    input:
    path(vcf)
    val(vep_cache)

    script:
    """
    vep \
        --input_file $vcf \
        --fasta $params.human_reference_fasta \
        --dir_cache $vep_cache 
    """

}
</code></pre> 
</div> 
<h2>Build a custom AMI to streamline the setup</h2> 
<p>To optimize your infrastructure deployment you can <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html">create a custom Amazon Machine Image (AMI)</a> with pre-installed Mountpoint for Amazon S3. This approach eliminates the need for repetitive downloading of the Mountpoint client during each EC2 instance launch. By packaging Mountpoint into your custom AMI, you can ensure faster instance startup times and standardize your deployment configuration across your compute environment.</p> 
<p>Create your custom AMI from the launch template through the AWS Management Console:</p> 
<ol> 
 <li>Navigate to the EC2 Dashboard</li> 
 <li>Locate the “Launch Templates” option in the left navigation pane</li> 
 <li>Select your Mountpoint launch template</li> 
 <li>Use the “Actions” dropdown</li> 
 <li>Choose “Launch instance from template.”</li> 
</ol> 
<p>Once your instance is running with Mountpoint for Amazon S3 properly installed and configured, create your custom AMI:</p> 
<ol> 
 <li>Select the instance in the EC2 console</li> 
 <li>Choose “Actions”</li> 
 <li>Select “Image and templates”</li> 
 <li>Choose “Create image” to create your AMI</li> 
</ol> 
<p>Now, update your launch template to remove the Mountpoint setup from the <em>User Data</em> section and specify your custom AMI in the <em>Application and OS Images</em> section.</p> 
<h2>Conclusion</h2> 
<p>In this post, we showed how to optimize Nextflow workflows on AWS Batch by using Mountpoint for Amazon S3 to stream reference data. This approach eliminates file staging bottlenecks for large static reference files in your genomic pipelines. To get started and learn more about mounting S3 buckets as a local file system, visit the Mountpoint for Amazon S3 <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/mountpoint.html">documentation</a> and <a href="https://github.com/awslabs/mountpoint-s3">Github repository</a>.</p>
