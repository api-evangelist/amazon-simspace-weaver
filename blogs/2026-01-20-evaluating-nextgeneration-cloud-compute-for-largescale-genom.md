---
title: "Evaluating next‑generation cloud compute for large‑scale genomic processing"
url: "https://aws.amazon.com/blogs/hpc/evaluating-next-generation-cloud-compute-for-large-scale-genomic-processing/"
date: "Tue, 20 Jan 2026 17:56:37 +0000"
author: "Manu Pillai"
feed_url: "https://aws.amazon.com/blogs/hpc/feed/"
---
<p><em>This post was contributed by Manu Pillai (AWS), Marissa E Powers PhD (AWS), Sean O’Dell (AstraZeneca), Gabriel Hernandez (AstraZeneca), Heejoon Jo (Illumina), Shyamal Mehtalia (Illumina), Joe Warren (AWS), and Natalia Jimenez PhD (AWS).</em></p> 
<p>Genomic research demands massive computational power to process and analyze DNA sequences that unlock insights into human health and disease. For pharmaceutical companies, processing millions of genomes efficiently and sustainably is a crucial factor to accelerate discovery and development of future therapies that have the potential to help people live longer, healthier lives.</p> 
<p>As cloud infrastructure evolves, many organizations face the challenge of migrating to newer, more powerful instances while maintaining research integrity and managing costs. When Amazon Elastic Compute Cloud (Amazon EC2) F1 instances approach end-of-service in 2025, organisations relying on Field-Programmable Gate Array (FPGA)-based instances to process genetic data may need a clear path forward that doesn’t compromise critical workloads.</p> 
<p>In this post, we describe a structured evaluation of Amazon EC2 F2 instances for accelerated genomics workloads, focusing on performance, cost, and output equivalency relative to earlier FPGA-based instance generations.</p> 
<h2>The challenge: future-proofing genomic infrastructure</h2> 
<p>AstraZeneca’s Centre for Genomics Research (CGR) integrates large-scale genomic and clinical data leveraging multi-omics approaches—genomics, transcriptomics, proteomics, and metabolomics—at population scale, making it one of the largest and most diverse datasets globally.</p> 
<p>The CGR use Illumina DRAGEN®<a href="#_ftn1" name="_ftnref1">[1]</a> (Dynamic Read Analysis for GENomics) workloads for processing Whole Exome Sequencing (WES) and Whole Genome Sequencing (WGS) data. With F1 instances approaching end-of-service, we evaluated F2 instances as the next-generation solution for continuing workloads without disruption.</p> 
<h2>The solution: comprehensive F1 to F2 migration testing</h2> 
<p>In collaboration with AstraZeneca and Illumina, AWS led extensive testing comparing F1 and F2 instance performance across multiple genomic workloads and AWS Regions.</p> 
<h3>Testing methodology</h3> 
<p>The team designed a comprehensive evaluation using:</p> 
<ul> 
 <li>Sample Analysis: 11 genomic samples (5 WES and 6 WGS) representing typical research workloads</li> 
 <li>Architecture: AWS Batch with separate compute queues for F1 and F2 instances</li> 
 <li>Illumina DRAGEN Versions: v3.7.8 for WGS processing, v4.3.6 for WES processing</li> 
 <li>Multi-Region Validation: Testing across three AWS regions to ensure global scalability</li> 
 <li>Performance Metrics: Runtime comparison and cost analysis for identical workloads</li> 
 <li>Validation: A formal method for validating genomic equivalency</li> 
</ul> 
<p>The architecture leveraged the <a href="https://www.nextflow.io/">Nextflow</a> workflow system with <a href="https://aws.amazon.com/batch/">AWS Batch</a> (a fully managed batch computing service) to orchestrate DRAGEN pipeline execution, enabling consistent testing conditions across both instance types.</p> 
<div class="wp-caption aligncenter" id="attachment_5491" style="width: 1034px;">
 <img alt="Figure 1 - This diagram shows the cloud testing environment used to compare F1 and F2 instance performance for genomic processing. Users submit DRAGEN workflows through Nextflow, which orchestrates separate AWS Batch compute environments for Whole Exome Sequencing (WES) and Whole Genome Sequencing (WGS) samples on F2.6xlarge instances. The standardized architecture enabled precise performance comparisons by maintaining identical workflow orchestration while only changing the underlying compute infrastructure, ensuring the 60% performance improvements and 70% cost reductions could be accurately attributed to F2 instances." class="size-large wp-image-5491" height="539" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2025/12/16/hpcblog-421-fig1-1024x539.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5491">Figure 1 – This diagram shows the cloud testing environment used to compare F1 and F2 instance performance for genomic processing. Users submit DRAGEN workflows through Nextflow, which orchestrates separate AWS Batch compute environments for Whole Exome Sequencing (WES) and Whole Genome Sequencing (WGS) samples on F2.6xlarge instances. The standardized architecture enabled precise performance comparisons by maintaining identical workflow orchestration while only changing the underlying compute infrastructure, ensuring the 60% performance improvements and 70% cost reductions could be accurately attributed to F2 instances.</p>
</div> 
<h2>Results: dramatic performance and cost improvements</h2> 
<p>The testing revealed significant improvements when migrating from F1 to F2 instances:</p> 
<p>Key Benefits</p> 
<ul> 
 <li>60% faster processing: Significant reduction in genomic analysis runtime</li> 
 <li>70% cost reduction: Substantial savings on compute costs</li> 
 <li>Sustainability improvement: Reduced energy use and associated carbon emissions by completing workloads more efficiently</li> 
 <li>Enhanced scalability: Improved performance across multiple AWS regions</li> 
 <li>Maintained accuracy: No compromise in research data integrity</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_5493" style="width: 1034px;">
 <img alt="Figure 2 - This chart compares F1.4xlarge and F2.6xlarge instance performance for six Whole Genome Sequencing samples, showing both runtime (in minutes) and cost per run (in dollars). F2 instances consistently deliver up to 60% faster processing times and 70% lower costs across all samples, demonstrating the substantial performance improvements and cost savings organizations can achieve when upgrading from F1 to F2 instances for genomic workloads. On Demand pricing for f1.4xlarge in eu-west-1 and f2.6xlarge in eu-west-2 as of 9/16/2025." class="size-large wp-image-5493" height="304" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2025/12/16/hpcblog-421-fig2-1024x304.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5493">Figure 2 – This chart compares F1.4xlarge and F2.6xlarge instance performance for six Whole Genome Sequencing samples, showing both runtime (in minutes) and cost per run (in dollars). F2 instances consistently deliver up to 60% faster processing times and 70% lower costs across all samples, demonstrating the substantial performance improvements and cost savings organizations can achieve when upgrading from F1 to F2 instances for genomic workloads. On Demand pricing for f1.4xlarge in eu-west-1 and f2.6xlarge in eu-west-2 as of 9/16/2025.</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_5492" style="width: 1034px;">
 <img alt="Figure 3 - This chart compares F1.4xlarge and F2.6xlarge instance performance for five Whole Exome Sequencing samples, showing both runtime (in minutes) and cost per run (in dollars). F2 instances consistently deliver up to 63% faster processing times and 74% lower costs across all samples, demonstrating the substantial performance improvements and cost savings organizations can achieve when upgrading from F1 to F2 instances for exome sequencing workloads. On Demand pricing for f1.4xlarge in eu-west-1 and f2.6xlarge in eu-west-2 as of 9/16/2025." class="size-large wp-image-5492" height="364" src="https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2025/12/16/hpcblog-421-fig3-1024x364.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-5492">Figure 3 – This chart compares F1.4xlarge and F2.6xlarge instance performance for five Whole Exome Sequencing samples, showing both runtime (in minutes) and cost per run (in dollars). F2 instances consistently deliver up to 63% faster processing times and 74% lower costs across all samples, demonstrating the substantial performance improvements and cost savings organizations can achieve when upgrading from F1 to F2 instances for exome sequencing workloads. On Demand pricing for f1.4xlarge in eu-west-1 and f2.6xlarge in eu-west-2 as of 9/16/2025.</p>
</div> 
<h2>Validating equivalency</h2> 
<p>To assess Illumina DRAGEN output equivalency across AWS F1 and F2 instance families, we evaluated the output for five samples processed on each instance type and compared outputs at three levels: pipeline metrics, command provenance, and variant calls. For like-for-like runs, we expect identical results across instances. In practice, every value and variant should match exactly, with differences limited to non-deterministic metadata such as timestamps, file paths, or sample labels.</p> 
<p>Command Provenance: To verify that the pipelines were invoked identically, we extracted the DRAGEN command-lines from the VCF headers and compared them using bash tools. The commands and flags matched exactly between F1 and F2 runs, ensuring the analysis was conducted with the same configuration and parameters.</p> 
<p>Metrics Comparison: We compared DRAGEN-generated metrics using standard bash utilities (e.g., diff) across the following files: gc_metrics, mapping_metrics, roh_metrics, sv_metrics, and vc_metrics. &nbsp;Across all five samples, the metrics were identical between F1 and F2 outputs except for the values we expect to vary (e.g., names, date/timestamp). This confirms that alignment, variant calling, and summary statistics are reproducible across both instance types.</p> 
<p>VCF Concordance: We evaluated variant-level concordance using bcftools isec, which performs set operations on VCF files by treating each file as a set of variant records. It computes intersections (variants present in both files) and differences (variants unique to one file), making it a practical way to quantify concordance between outputs. In our workflow, we ran bcftools isec on paired VCFs (F1 vs F2) for each sample. The tool writes out separate files corresponding to the following:</p> 
<ul> 
 <li>Variants unique to the first VCF (e.g., 0000.vcf)</li> 
 <li>Variants unique to the second VCF (e.g., 0001.vcf)</li> 
 <li>Variants common to both VCFs (the intersection, e.g., 0002.vcf, and 0003.vcf)</li> 
</ul> 
<p>We then counted variants in each output to measure overlap and differences, excluding header lines that begin with # or ##. Under full equivalence, the unique outputs should be empty, and the intersection files should contain all variants, so intersection record counts match the originals.</p> 
<pre><code class="lang-bash"># Run bcftools isec

$&gt; bcftools isec -p results sample1_f1.vcf.gz sample1_f2.vcf.gz

# Count variant records (exclude header lines)

$&gt; grep -c -v "^#" results/*.vcf
0000.vcf:0
0001.vcf:0
0002.vcf:299131
0003.vcf:299131</code></pre> 
<p>Across all five samples, bcftools isec showed that the intersection contained all variants, and the unique sets were empty, indicating no variant-level differences between F1 and F2 outputs. In other words, the VCFs were fully concordant.</p> 
<p>In conclusion, under identical DRAGEN configurations, Amazon EC2 F1 and F2 instances produce equivalent&nbsp; outputs at both the metrics and variant levels. Metrics differed only in expected, non-deterministic metadata (timestamps and sample labels), and bcftools isec showed that variant intersections contained all calls with no unique differences. These results support running DRAGEN genomic pipelines interchangeably on F1 and F2.</p> 
<h2>The impact: accelerating genomic discovery</h2> 
<p>The data suggest that successful migration includes advantages that extend beyond simpler infrastructure upgrades. The immediate benefits include establishing a clear, validated migration path from F1 to F2 instances, ensuring business continuity and achieving significant cost and CO2 emissions optimization for large-scale genomic workloads.</p> 
<h2>Conclusion</h2> 
<p>The results demonstrate how cloud infrastructure evolution can drive efficiency improvements in genomic research capabilities. The 60% performance improvement, 70% cost reduction, as well as improved sustainability achieved through F2 instance migration provides compelling data that may be useful for organizations looking to optimize their genomic processing workloads.</p> 
<p>For genomics teams currently using F1 instances, this validation study offers confidence in migrating to F2 instances while maintaining research integrity and achieving substantial operational benefits.</p> 
<p>To learn more about AWS F2 instances for genomics workloads, visit <a href="https://aws.amazon.com/health/genomics/">Genomics on AWS</a>. For DRAGEN implementation guidance, explore the <a href="https://www.illumina.com/products/by-type/informatics-products/dragen-secondary-analysis.html">Illumina DRAGEN product information page</a>.</p> 
<p><a href="#_ftnref1" name="_ftn1">[1]</a> DRAGEN® is a registered trademark of Illumina Inc.</p>
