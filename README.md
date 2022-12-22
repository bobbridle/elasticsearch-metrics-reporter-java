# Metrics Elasticsearch Reporter (Patched)

## Introduction

This is a forked version of the [Metrics Elasticsearch Reporter](https://github.com/elastic/elasticsearch-metrics-reporter-java) to make it compatible with Elasticsearch 7.10+. 

The [Metrics Elasticsearch Reporter](https://github.com/elastic/elasticsearch-metrics-reporter-java) is a reporter for the excellent [Metrics library](http://metrics.dropwizard.io/), similar to the [Graphite](http://metrics.dropwizard.io/3.1.0/manual/graphite/) or [Ganglia](http://metrics.dropwizard.io/3.1.0/manual/ganglia/) reporters, except that it reports to an Elasticsearch server.

## Compatibility

| metrics-elasticsearch-reporter-patched | elasticsearch | Release date |
|----------------------------------------|---------------|:------------:|
| 2.3.0                                  | 7.10+         |  22-12-2022  |

## Patched changes (Git Patch)

Subject: [PATCH] ElasticSearch 7.10+ compatibility
---
===================================================================
diff --git a/src/main/java/org/elasticsearch/metrics/ElasticsearchReporter.java b/src/main/java/org/elasticsearch/metrics/ElasticsearchReporter.java
--- a/src/main/java/org/elasticsearch/metrics/ElasticsearchReporter.java	(revision 4018eaef6fc7721312f7440b2bf5a19699a6cb56)
+++ b/src/main/java/org/elasticsearch/metrics/ElasticsearchReporter.java	(date 1671593544459)
@@ -288,7 +288,7 @@
}

         try {
-            HttpURLConnection connection = openConnection("/_bulk", "POST");
+            HttpURLConnection connection = openConnection("/" + currentIndexName + "/_bulk", "PUT");
             if (connection == null) {
                 LOGGER.error("Could not connect to any configured elasticsearch instances: {}", Arrays.asList(hosts));
                 return;
@@ -350,7 +350,7 @@
* Execute a percolation request for the specified metric
*/
private List<String> getPercolationMatches(JsonMetric jsonMetric) throws IOException {
-        HttpURLConnection connection = openConnection("/" + currentIndexName + "/" + jsonMetric.type() + "/_percolate", "POST");
+        HttpURLConnection connection = openConnection("/" + currentIndexName /*+ *//*"/" + jsonMetric.type() + "/_percolate"*/, "PUT");
         if (connection == null) {
             LOGGER.error("Could not connect to any configured elasticsearch instances for percolation: {}", Arrays.asList(hosts));
             return Collections.emptyList();
@@ -412,7 +412,7 @@
private HttpURLConnection createNewConnectionIfBulkSizeReached(HttpURLConnection connection, int entriesWritten) throws IOException {
if (entriesWritten % bulkSize == 0) {
closeConnection(connection);
-            return openConnection("/_bulk", "POST");
+            return openConnection("/" + currentIndexName + "/_bulk", "PUT");
         }

         return connection;
@@ -422,7 +422,7 @@
* serialize a JSON metric over the outputstream in a bulk request
*/
private void writeJsonMetric(JsonMetric jsonMetric, ObjectWriter writer, OutputStream out) throws IOException {
-        writer.writeValue(out, new BulkIndexOperationHeader(currentIndexName, jsonMetric.type()));
+        writer.writeValue(out, new BulkIndexOperationHeader(currentIndexName,null));
         out.write("\n".getBytes());
         writer.writeValue(out, jsonMetric);
         out.write("\n".getBytes());
@@ -438,6 +438,8 @@
try {
URL templateUrl = new URL("http://" + host  + uri);
HttpURLConnection connection = ( HttpURLConnection ) templateUrl.openConnection();
+                connection.setRequestProperty("Content-Type", "application/json");
+                connection.setRequestProperty("Accept", "application/json");
                 connection.setRequestMethod(method);
                 connection.setConnectTimeout(timeout);
                 connection.setUseCaches(false);
Index: pom.xml
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/pom.xml b/pom.xml
--- a/pom.xml	(revision 4018eaef6fc7721312f7440b2bf5a19699a6cb56)
+++ b/pom.xml	(date 1671679628814)
@@ -2,13 +2,13 @@
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

     <groupId>org.elasticsearch</groupId>
-    <artifactId>metrics-elasticsearch-reporter</artifactId>
-    <version>2.3.0-SNAPSHOT</version>
+    <artifactId>metrics-elasticsearch-reporter-patched</artifactId>
+    <version>2.3.0</version>

     <properties>
         <lucene.version>5.5.0</lucene.version>
         <elasticsearch.version>2.3.1</elasticsearch.version>
-        <jackson.version>2.7.3</jackson.version>
+        <jackson.version>2.9.0</jackson.version>
         <randomized.testrunner.version>2.3.3</randomized.testrunner.version>
     </properties>

@@ -18,11 +18,11 @@
<description>Reporter for the metrics library which reports into Elasticsearch</description>
<inceptionYear>2013</inceptionYear>

-    <scm>
-        <connection>scm:git:git@github.com:elasticsearch/elasticsearch-metrics-reporter-java.git</connection>
-        <developerConnection>scm:git:git@github.com:elasticsearch/elasticsearch-metrics-reporter-java.git</developerConnection>
-        <url>http://github.com/elasticsearch/elasticsearch-metrics-reporter</url>
-    </scm>
+<!--    <scm>-->
+<!--        <connection>scm:git:git@github.com:elasticsearch/elasticsearch-metrics-reporter-java.git</connection>-->
+<!--        <developerConnection>scm:git:git@github.com:elasticsearch/elasticsearch-metrics-reporter-java.git</developerConnection>-->
+<!--        <url>http://github.com/elasticsearch/elasticsearch-metrics-reporter</url>-->
+<!--    </scm>-->

     <licenses>
         <license>
@@ -32,17 +32,6 @@
</license>
</licenses>

-    <parent>
-        <groupId>org.sonatype.oss</groupId>
-        <artifactId>oss-parent</artifactId>
-        <version>9</version>
-    </parent>
-
-    <issueManagement>
-        <system>GitHub</system>
-        <url>https://github.com/elasticsearch/elasticsearch-metrics-reporter-java/issues/</url>
-    </issueManagement>
-
   <build>
       <plugins>
           <plugin>
@@ -84,7 +73,7 @@
<dependency>
<groupId>io.dropwizard.metrics</groupId>
<artifactId>metrics-core</artifactId>
-            <version>3.1.2</version>
+            <version>4.0.0</version>
         </dependency>
         <dependency>
             <groupId>com.fasterxml.jackson.core</groupId>
Index: src/main/java/org/elasticsearch/metrics/MetricsElasticsearchModule.java
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/src/main/java/org/elasticsearch/metrics/MetricsElasticsearchModule.java b/src/main/java/org/elasticsearch/metrics/MetricsElasticsearchModule.java
--- a/src/main/java/org/elasticsearch/metrics/MetricsElasticsearchModule.java	(revision 4018eaef6fc7721312f7440b2bf5a19699a6cb56)
+++ b/src/main/java/org/elasticsearch/metrics/MetricsElasticsearchModule.java	(date 1671593544483)
@@ -256,9 +256,9 @@
if (bulkIndexOperationHeader.index != null) {
json.writeStringField("_index", bulkIndexOperationHeader.index);
}
-            if (bulkIndexOperationHeader.type != null) {
-                json.writeStringField("_type", bulkIndexOperationHeader.type);
-            }
+           // if (bulkIndexOperationHeader.type != null) {
+                json.writeStringField("_type", "counter");
+           // }
             json.writeEndObject();
             json.writeEndObject();
         }
Index: src/main/java/org/elasticsearch/metrics/JsonMetrics.java
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/src/main/java/org/elasticsearch/metrics/JsonMetrics.java b/src/main/java/org/elasticsearch/metrics/JsonMetrics.java
--- a/src/main/java/org/elasticsearch/metrics/JsonMetrics.java	(revision 4018eaef6fc7721312f7440b2bf5a19699a6cb56)
+++ b/src/main/java/org/elasticsearch/metrics/JsonMetrics.java	(date 1671593544471)
@@ -56,7 +56,7 @@

         @Override
         public String toString() {
-            return String.format("%s %s %s", type(), name, timestamp);
+            return String.format("%s %s %s %s", type(), name, timestamp,value);
         }

         public abstract String type();

