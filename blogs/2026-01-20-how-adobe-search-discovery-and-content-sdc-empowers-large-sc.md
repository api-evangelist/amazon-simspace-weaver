---
title: "How Adobe Search, Discovery, and Content (SDC) Empowers Large Scale Image (AI/ML) Inferencing Utilizing AWS Batch"
url: "https://aws.amazon.com/blogs/hpc/how-adobe-search-discovery-and-content-sdc-empowers-large-scale-image-ai-ml-inferencing-utilizing-aws-batch/"
date: "Tue, 20 Jan 2026 15:53:01 +0000"
author: "Harish Suvarna"
feed_url: "https://aws.amazon.com/blogs/hpc/feed/"
---
<p><em>This post was contributed by Harish Suvarna, Alec Ngo, and Suryanarayana Murthy Boddu at Adobe, and Mark Azadpour, Yoginder Sethi at AWS.</em></p> 
<p>Adobe empowers everyone everywhere to imagine, create, and bring any digital experience to life. With over 29 billion assets generated using Adobe Firefly and a vast Adobe Stock ecosystem, the ability to search and discover relevant images or videos at scale is critical.</p> 
<p>Achieving this at Adobe’s scale comes with complex technical requirements: running AI models that process hundreds of millions of assets, managing thousands of parallel inference jobs, and ensuring GPUs are fully utilized without breaking cost efficiency. Meeting these challenges requires an infrastructure that can handle massive data throughput, scale elastically, and minimize operational overhead.</p> 
<p>In this post, we will share how the Adobe SDC team tackled these challenges and why AWS Batch—combined with GPU accelerated Amazon EC2 Instances—became a strong fit for our large-scale distributed inference workloads. More importantly, we’ll highlight the lessons we learned and the practices that helped us make inference very efficient.</p> 
<h2>The opportunity</h2> 
<p>Image assets from multiple sources power content retrieval, discovery and recommendations across various Adobe products—such as Adobe Acrobat, Adobe Express, Adobe Firefly and more—and are surfaced through semantic search. Embedding models are constantly evolving, which makes&nbsp;periodic re-computation of the embeddings for such huge data set(s) with hundreds of millions of assets increasingly critical for state-of-the-art search ranking and relevancy. &nbsp;As search features and experiences continue to evolve, with several experiments and evaluations across the development cycle, recomputing and updating embeddings becomes repetitive.</p> 
<p>Computing image embeddings is similar to workflows for text, video, and audio and involves loading images into memory, running inference with Adobe’s proprietary machine learning models, and storing the results in internal systems that power semantic search, RAG, and other capabilities. Given the volume of the data, this process can only be implemented as a massively parallel process rather than sequential processing. &nbsp;AWS Batch, a fully managed service that simplifies running batch computing workloads, helped to achieve our goals.</p> 
<p>Some of the key challenges included:</p> 
<ul> 
 <li><strong>Model and data diversity</strong>: Different models and dataset sizes require different GPU architectures to maximize performance and speed. So, our pipelines need to be flexible to use different types of EC2 instances</li> 
 <li><strong>High operational overhead</strong>: Building and maintaining our own container orchestration system (such as EKS) to execute these <em>ad hoc</em> inference jobs would involve&nbsp;significant engineering effort, including managing servers, auto-scaling groups, and load balancers. Instead, a simple ephemeral solution is more appropriate.</li> 
 <li><strong>Failure detection and recovery</strong>: Even with complete automation of cluster setup, failure detection and retry mechanisms require additional business logic.</li> 
 <li><strong>Data sharding complexity</strong>: Efficiently partitioning data and assigning work to each node in the data lake becomes increasingly challenging at scale.</li> 
 <li><strong>Long processing times</strong>: Sequential processing can take weeks to complete for large workloads, severely limiting iteration speed.</li> 
</ul> 
<h2>The solution</h2> 
<p>When designing solutions, we aimed to achieve:</p> 
<ul> 
 <li><strong>Immediate access to resources</strong>: Provide seamless access to GPUs across multiple instance types (e.g., g6e, g5), selected based on model requirements.</li> 
 <li><strong>Efficient resource utilization</strong>: Ensure resources allocated are efficiently and effectively used.</li> 
 <li><strong>Consistent job submission</strong>: Offer a standardized interface for job execution, regardless of parallelization level or EC2 instance type.</li> 
 <li><strong>Cost optimization</strong>: Leverage a “pay-as-you-go” model and dynamically scale resources up or down based on demand. Minimize per-job costs to deliver the best possible value.</li> 
 <li><strong>User experience</strong>: Democratize massive batch processing through a fully self-serve experience, empowering users to run jobs without deep infrastructure knowledge.</li> 
</ul> 
<h2>Data Preparation and Distributed Inferencing</h2> 
<p>Efficient data loading is critical to maximize GPU utilization during large-scale inference. A common bottleneck is that data is either not loaded quickly enough into GPUs, or the pipeline becomes unbalanced, leading to underutilized hardware.</p> 
<p>For a different use case, Adobe Research customized the PyTorch Dataloader to stream samples directly from Amazon S3 into AWS Batch containers at a node level. Within each container, CPUs handle preprocessing—such as decoding and converting samples into tensors—before delivering well-formed batches to GPUs for inference. If a node has 8 GPUs, 8 dataloaders are created, each paired with its own model engine. While they all point to the same dataset, each data loader uses its assigned rank and the number of dataloaders the host is spinning up, also known as world rank, to know exactly which shard of the dataset to process, avoiding overlap and ensuring balanced throughput.</p> 
<div class="wp-caption aligncenter" id="attachment_5557" style="width: 1034px;">
 <img alt="Figure 1: Dataloader System for Inferencing" class="size-large wp-image-5557" height="651" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/01/20/HPCBlog-394-f1-1024x651.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5557">Figure 1: Dataloader System for Inferencing.</p>
</div> 
<p>When extending the process to multiple distributed nodes, it is important for each node to be given a distinct dataset partition to process. One approach is to download the entire dataset per node and rely on dataloader rank/world size to handle partitioning. However, given the scale of the data, this would require significant extra disk usage. This is where we leveraged AWS Batch: We separated the dataset preparation and converted it into its own <strong>AWS Batch job</strong>. This preprocessing job runs multiple workers to shard the S3 bucket and compress object addresses into compact binary files.</p> 
<p>During inference, each node only needs to download its assigned binary shard—based on its AWS_BATCH_JOB_NODE_INDEX—and to feed that to its data loaders. This reduces disk overhead, eliminates redundant data transfers, and ensures that every node processes a unique subset of the dataset. Together, these techniques keep GPUs well-fed with batches and maintain high efficiency across thousands of concurrent jobs. This stage was also achieved by AWS Batch service, and we leveraged Batch’s EC2 GPU instance type support capability.</p> 
<div class="wp-caption aligncenter" id="attachment_5559" style="width: 1034px;">
 <img alt="Figure 2: Inference whole system Diagram" class="size-large wp-image-5559" height="234" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/01/20/HPCBlog-394-f2-1024x234.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5559">Figure 2: Inference whole system Diagram.</p>
</div> 
<h2>How AWS Batch Supports our Solution Goals and Solved Scaling Challenges</h2> 
<p>AWS Batch provides a managed architecture that removes much of the operational burden of scaling large-scale inference workloads. It enables us to run tens of thousands of concurrent jobs without maintaining complex infrastructure, while benefiting from consistent logging, monitoring, access controls, and file system integration.</p> 
<p>The following are some of the elements of the AWS Batch parallel processing.</p> 
<p><u>Compute environment</u>: This is where we define the EC2 instance type, launch template, and the minimum/maximum number of nodes. AWS Batch provisions an Auto Scaling Group (ASG) for GPU nodes and deploys containers to these EC2 nodes based on their availability and health. This abstraction means we rarely touch low-level infrastructure. Scaling is automatic—if job volume increases or decreases, the ASG adjusts node count within the defined limits, ensuring we never pay for idle capacity.</p> 
<p><u>Job queue</u>: This determines the order in which jobs are executed against the associated Compute Environment. For example, if queue A is mapped to compute environment (CE) A, jobs in that queue will be dispatched based on available GPU capacity and queue priority. The job queue also enforces high-level execution rules, such as terminating jobs that exceed timeouts, fail due to insufficient GPU availability, or encounter other resource constraints.</p> 
<p><u>Job definition</u>: The job’s execution blueprint, like a <strong>docker run</strong> command. This specifies the Docker image, required GPU/CPU resources, disk mounts, and any environment variables or runtime configurations needed for the container to run on the EC2 host. It also defines the number of <em>job attempts</em>, or how many times to try to restart the container in case of failure.</p> 
<p><u>Job</u>: The process of submitting a job definition into a job queue. This triggers the provisioning of the required infrastructure. Each node in a multi-node job is automatically assigned two key environment variables—AWS_BATCH_JOB_NODE_INDEX and AWS_BATCH_JOB_NUM_NODES—used to shard data so that each node processes its specific portion of the workload.</p> 
<p>When a job is submitted to AWS Batch, every job node receives the same environment variables, including an S3 URI where all data is stored. The EC2 instance from the ASG pulls the Docker image containing our inference model and logic. Once the container is up and healthy, it uses the node’s AWS_BATCH_JOB_NODE_INDEX and AWS_BATCH_JOB_NUM_NODES values to determine exactly which shard of data it should download and process. After the inference step completes, results are immediately stored in the search index. The container then exits with a success code and terminates, freeing the EC2 host. If additional jobs are waiting in the queue, the host picks up the next job; otherwise, the ASG scales down to release unused resources. By default, each job index also creates an AWS CloudWatch log stream, allowing us to observe container activity in real time and debug issues when needed.</p> 
<div class="wp-caption aligncenter" id="attachment_5558" style="width: 1034px;">
 <img alt="Figure 3 – AWS Batch data flow diagram used with Adobe solution." class="size-large wp-image-5558" height="354" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2026/01/20/HPCBlog-394-f3-1024x354.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5558">Figure 3 – AWS Batch data flow diagram used with Adobe solution.</p>
</div> 
<h2>Results</h2> 
<p>By redesigning a sequential workflow into a <strong>multi-step sharding and distributed parallel compute pipeline</strong>, and by implementing an efficient <strong>onload/offload mechanism for data to GPUs</strong>, we achieved <strong>10x faster execution</strong>. From our monitoring dashboards, we often see our GPUs now consistently run at <strong>100% utilization</strong>, delivering extremely high throughput and maximizing hardware efficiency.</p> 
<p>AWS Batch allowed us to support this at scale by provisioning ephemeral compute with no fixed infrastructure or standing costs. AWS Batch handles the operational heavy lifting, and its flexibility to support both CPU and GPU workloads has saved us substantial engineering effort while reducing cloud spend.</p> 
<p>In one of our largest runs—computing image embeddings for Adobe Stock—we reduced compute time from <strong>30 days to under 20 hours</strong>, while realizing a <strong>95%+ reduction in cost</strong>, covering not only compute but also engineering investment. This capability empowers our engineering team to accelerate production workloads and run large-scale experiments quickly and cost-effectively.</p> 
<h2>Lessons Learned</h2> 
<h3>Data Organization</h3> 
<ol> 
 <li>Efficient Data Sharding:<br /> To process a large number of assets efficiently, pre-organizing data into smaller, manageable shards is critical. Without this, even simple operations like listing assets in S3 can take hours or may hang entirely. By organizing the data into smaller shards, we enabled parallel nodes to consume the data more effectively.</li> 
 <li>Handling S3 Rate Limits:<br /> After organizing the data, parallel downloading by multiple nodes initially resulted in rate limit errors (HTTP 429). To address this:</li> 
</ol> 
<ul> 
 <li>We collaborated with the AWS S3 team to increase the throughput of the S3 bucket, which significantly reduced the rate limit errors.</li> 
 <li>Despite these measures, occasional 429 errors persisted. To mitigate this: 
  <ul> 
   <li>We implemented immediate retries within the same processing loop.</li> 
   <li>We added a final retry mechanism at the end of the processing. By this stage, each node was making fewer calls to S3, reducing the likelihood of 429 errors.</li> 
  </ul> </li> 
</ul> 
<ol start="3"> 
 <li>Batch Processing on GPUs:<br /> When working with GPUs, processing data in optimal batches is mandatory. This applies to preprocessing, processing, and post-processing. This optimal batch size varies across applications. We experimented with various batch sizes before the big run and found an optimal size. Batch processing saved significant GPU time and reduced the overall processing time.</li> 
 <li>Data Localization:<br /> We ensured that the data resided in the same AWS region as the batch computing clusters. This approach:</li> 
</ol> 
<ul> 
 <li>Avoided cross-region data transfer charges, which can be substantial.</li> 
 <li>Reduced latency and improved the overall operation time.</li> 
</ul> 
<h3>Compute</h3> 
<ol start="5"> 
 <li>GPU Availability Challenges:<br /> Due to the rising demand for ML computing, obtaining a high number of GPUs on demand is challenging. To address this:</li> 
</ol> 
<ul> 
 <li>We created compute definitions for various GPU instance families such as G5s, G6es, and G6s.</li> 
 <li>We leveraged hybrid compute definitions in AWS Batch to maximize flexibility.</li> 
</ul> 
<ol start="6"> 
 <li>Throughput Optimization:<br /> Understanding the throughput of a single pod was essential. Based on this:</li> 
</ol> 
<ul> 
 <li>We defined the minimum and maximum number of vCPUs/GPUs required.</li> 
 <li>This ensured that even if the full compute capacity was unavailable, AWS Batch could start processing with the available resources and scale up as more compute became available.</li> 
</ul> 
<h3>Key Takeaways:</h3> 
<ol> 
 <li>Data Organization: Proper sharding and localization of data are critical for efficient processing and cost optimization.</li> 
 <li>Error Handling: Implementing robust retry mechanisms helps mitigate rate limit errors.</li> 
 <li>Batch Processing: Processing data in batches is essential for GPU efficiency.</li> 
 <li>Compute Flexibility: Adapting to available compute resources ensures uninterrupted processing and scalability.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>By integrating AWS Batch with Amazon ECS and GPU-based EC2 instances, Adobe has significantly enhanced its &nbsp;asset &nbsp;processing (images, video) infrastructure while extending support to additional content types, including text and video. The result is <strong>unmatched scalability, cost-efficiency, and operational simplicity</strong>—enabling faster execution of massive workloads and accelerating innovation. For organizations aiming to scale high-performance computing and speed up time-to-market, AWS Batch offers a compelling path to optimal total cost of ownership (TCO).</p>
