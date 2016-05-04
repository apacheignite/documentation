In classical MapReduce paradigm we have a well-defined and finite set of jobs, which is known on the Map step and doesn't change throughout all computation run. But what if we have a stream of jobs instead? In this case we can still run MapReduce using Ignite's continuous mapping facility. With continuous mapping, the jobs can be generated on-the-fly when computation is already running. The newly generated jobs are processed by worker nodes as usual, and the reducer receives results just like in normal MapReduce.
[block:api-header]
{
  "type": "basic",
  "title": "Running Continuously Mapped Tasks"
}
[/block]
To use continuous mapping within your task, you need to inject `TaskContinuousMapperResource` resource into a task instance:
[block:code]
{
  "codes": [
    {
      "code": "@TaskContinuousMapperResource\nprivate TaskContinuousMapper mapper;",
      "language": "java"
    }
  ]
}
[/block]
After this, new jobs can be generated asynchronously and added to a currently running computation using `send()` methods from `TaskContinuousMapper` interface:
[block:code]
{
  "codes": [
    {
      "code": "mapper.send(new ComputeJobAdapter() {\n    @Override public Object execute() {\n        System.out.println(\"I'm a continuously-mapped job!\");\n \n        return null;\n    }\n});",
      "language": "java"
    }
  ]
}
[/block]
For continuous mapping, there are several constraints that you need to be aware of:
  *  If you initially return null from `ComputeTask.map()` method, you should send at least one job with continuous mapper before returning.
  * Continuous mapper can not be used after `ComputeTask.result()` method returned the `REDUCE` policy.
  * If `ComputeTask.result()` method returned the `WAIT` policy and all jobs are finished, then task will go to `Reduce` step and continuous mapper can not be used any more.

In other respects, the computation logic is the same as in normal MapReduce, described in [MapReduce](doc:map-reduce) chapter.
[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
The following example performs Web site analysis based on the images it contains. The Web search engine scans through all images available on the site, and the MapReduce engine, implemented with Ignite, determines the category ("meaning") of each image based on some image-analysis algorithm. The results of image analysis are then reduced to determine the site category. This example is good for demonstrating continuous mapping, because Web search is a continuous process, with which MapReduce can run in parallel and analyze new Web search results (new image files) as they come. This can save sufficient amount of time, compared to traditional MapReduce, where we would first wait for all Web search results (a full set of images) and only then map them to nodes, analyze, and reduce results.
Suppose we have a Crawler interface from some library for scanning site images, CrawlerListener interface for getting results of asynchronous background scanning, and an Image interface representing a single image file (the class names are intentionally made short to simplify reading):
[block:code]
{
  "codes": [
    {
      "code": "interface Crawler {\n    ...\n    public Image findNext();\n \n    public void findNextAsync(CrawlerListener listener);\n \n    ...\n}\n \ninterface CrawlerListener {\n    public void onImage(Crawler c, Image img) throws Exception;\n}\n \ninterface Image {\n    ...\n \n    public byte[] getBytes();\n \n    ...\n}",
      "language": "java"
    }
  ]
}
[/block]
Suppose we also have an ImageAnalysisJob that implements `ComputeJob` with logic for analyzing image:
[block:code]
{
  "codes": [
    {
      "code": "class ImageAnalysisJob implements ComputeJob, Externalizable {\n    ...\n \n    public ImageAnalysisJob(byte[] imgBytes) {\n        ...\n    }\n \n    @Nullable @Override public Object execute() throws IgniteException {\n        // Image analysis logic (returns some information \n        // about the image content: category, etc.).\n        ...\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
Given all the above, here is how we could run Web search and analysis in parallel using **continuous mapping**:
[block:code]
{
  "codes": [
    {
      "code": "enum SiteCategory {\n    ...\n}\n \n// Instantiate a Web search engine for a particular site.\nCrawler crawler = CrawlerFactory.newCrawler(siteUrl);\n \n// Execute a continuously-mapped task.\nSiteCategory result = ignite.compute().execute(new ComputeTaskAdapter<Crawler, SiteCategory>() {\n    // Interface for continuous mapping (injected on task instantiation).\n    @TaskContinuousMapperResource\n    private TaskContinuousMapper mapper;\n \n    // Map step.\n    @Nullable @Override\n    public Map<? extends ComputeJob, ClusterNode> map(List<ClusterNode> nodes, @Nullable Crawler c) throws IgniteException {\n        assert c != null;\n \n        // Find a first image synchronously to submit an initial job.\n        Image img = c.findNext();\n \n        if (img == null)\n            throw new IgniteException(\"No images found on the site.\");\n \n        // Submit an initial job.\n        mapper.send(new ImageAnalysisJob(img.getBytes()));\n \n        // Now start asynchronous background Web search and\n        // submit new jobs as search results come. This call\n        // will return immediately.\n        c.findNextAsync(new CrawlerListener() {\n            @Override public void onImage(Crawler c, Image img) throws Exception {\n                if (img != null) {\n                    // Submit a new job to analyse image file.\n                    mapper.send(new ImageAnalysisJob(img.getBytes()));\n \n                    // Move on with search.\n                    c.findNextAsync(this);\n                }\n            }\n        });\n \n        // Initial job was submitted, so we can return \n        // empty mapping.\n        return null;\n    }\n \n    // Reduce step.\n    @Nullable @Override public SiteCategory reduce(List<ComputeJobResult> results) throws IgniteException {\n        // At this point Web search is finished and all image \n        // files are analysed. Here we execute some logic for\n        // determining site category based on image content\n        // information.\n        return defineSiteCategory(results);\n    }\n}, crawler);",
      "language": "java"
    }
  ]
}
[/block]