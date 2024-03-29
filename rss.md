---
layout: none
title: "The example of a malformed rss item"
permalink: /rss/
---

<rss version="2.0"
     xml:base="https://developers.redhat.com/"
     xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <atom:link href="https://svgol.github.io/rss" rel="self" type="application/rss+xml"/>
    <title>svgol' test feed</title>
    <link>https://https://svgol.github.io/rss</link>
    <description>
      Stories and tutorials on the latest technologies in cloud application development.
    </description>
    <language>en</language>
    <lastBuildDate>Thu, 20 Jan 2022 07:00:00 +0000</lastBuildDate>
    <sy:updatePeriod>hourly</sy:updatePeriod>
    <sy:updateFrequency>1</sy:updateFrequency>
    <item>
      <title>How to connect Prometheus to OpenShift Streams for Apache Kafka</title>
      <link>https://developers.redhat.com/articles/2021/12/17/how-connect-prometheus-openshift-streams-apache-kafka</link>
      <description>
<p>You've always been able to view some metrics for <a href="https://developers.redhat.com/products/red-hat-openshift-streams-for-apache-kafka/overview">Red Hat OpenShift Streams for Apache Kafka</a> in the UI or access them via the API. We recently added a feature to OpenShift Streams for Apache Kafka that exports these metrics to <a href="https://prometheus.io">Prometheus</a>, a system monitoring and alerting toolkit. Connecting these services lets you view your exported metrics alongside your other Kafka cluster metrics.</p>

<p>This article shows you how to set up OpenShift Streams for Apache Kafka to export metrics to Prometheus. The example configuration includes integration with <a href="https://grafana.com">Grafana</a>, a data visualization application that is often used with Prometheus. Almost all observability products and services support Prometheus, so you can easily adapt what you learn in this article to your observability stack.</p>

<h2>Guides and prerequisites</h2>

<p>We will use a number of guides for this configuration; You can open them now or use the links in each section:</p>

<ul><li><a href="https://access.redhat.com/documentation/en-us/openshift_dedicated/4/html/creating_a_cluster/index">OpenShift Dedicated: Creating a cluster</a></li>
	<li><a href="https://docs.openshift.com/dedicated/identity_providers/config-identity-providers.html#config-github-idp_config-identity-providers">Configuring a GitHub identity provider</a></li>
	<li><a href="https://access.redhat.com/documentation/en-us/openshift_dedicated/4/html/administering_your_cluster/osd-admin-roles">OpenShift Dedicated: Managing administration roles and users</a></li>
	<li><a href="https://access.redhat.com/documentation/en-us/red_hat_openshift_streams_for_apache_kafka/1/guide/f351c4bd-9840-42ef-bcf2-b0c9be4ee30a">Getting started with Red Hat OpenShift Streams for Apache Kafka</a></li>
</ul><p>We will also use these examples from the Prometheus and Grafana projects, respectively:</p>

<ul><li><a href="https://github.com/prometheus-operator/prometheus-operator/tree/main/example/additional-scrape-configs">Additional scrape configs for Prometheus</a></li>
	<li><a href="https://github.com/grafana-operator/grafana-operator/blob/master/deploy/examples/GrafanaWithIngressHost.yaml">GrafanaWithIngressHost.yaml</a></li>
</ul><h2>Set up your Kubernetes cluster</h2>

<p>To begin, you need to set up a <a href="https://developers.redhat.com/topics/kubernetes">Kubernetes</a> cluster to run Prometheus and Grafana. The example in this article will use <a href="https://cloud.redhat.com/products/dedicated/?intcmp=701f20000012jmYAAQ">Red Hat OpenShift Dedicated</a>.</p>

<p>Start by following the <a href="https://access.redhat.com/documentation/en-us/openshift_dedicated/4/html/creating_a_cluster/index">OpenShift Dedicated guide</a> to creating a Customer Cloud Subscription cluster on Amazon Web Services. Then, follow the instruction to <a href="https://docs.openshift.com/dedicated/identity_providers/config-identity-providers.html#config-github-idp_config-identity-providers">configure a GitHub identity provider</a> so that you can use your GitHub ID to log in to <a href="https://developers.redhat.com/openshift">Red Hat OpenShift</a>. (Other options are available, but we're using these methods for the example.) Once you've got things configured, your OpenShift Cluster Manager should look like the screenshot in Figure 1.</p>

<div class="rhd-c-figure">
  <article class="media media--type-image media--view-mode-article-content"><div class="field field--name-image field--type-image field--label-hidden field__items">
              <a href="https://developers.redhat.com/sites/default/files/prometheus-fig1.png" data-featherlight="image"><img src="https://developers.redhat.com/sites/default/files/styles/article_floated/public/prometheus-fig1.png?itok=B6P_G0hA" width="600" height="147" alt="The OpenShift Cluster Manager with GitHub configured as an identity provider." typeof="Image" /></a>

          </div>
    <div class="field field--name-field-caption field--type-string field--label-hidden field__items">
                <div class="rhd-c-caption field__item">
        Figure 1. Configure GitHub as an identity provider for logging in to the OpenShift Dedicated console.
            </div>
          </div>

      </article></div>
<p>Finally, give your GitHub user the <code>cluster-admin</code> role by following the OpenShift Dedicated guide to <a href="https://access.redhat.com/documentation/en-us/openshift_dedicated/4/html/administering_your_cluster/osd-admin-roles">managing administration roles and users</a>. Note that the Prometheus examples assume you have admin permissions on the cluster. You'll need to use your GitHub username as the principal here.</p>

<p>Now, you can log in to the console using the <strong>Open Console</strong> button.</p>

<h2>Install Prometheus and Grafana</h2>

<p>You can install Prometheus on OpenShift Dedicated via the OperatorHub. Just navigate to <strong>Operators -> OperatorHub</strong>, filter for Prometheus, click <strong>Install</strong>, and accept the defaults. You can validate that it's installed in the <strong>Installed Operators</strong> list. Once that's done, do the same for Grafana.</p>

<p><strong>Note</strong>: You might have command-line muscle memory and prefer to use <code>kubectl</code> with a Kubernetes cluster. If you want to take this route, switch to a terminal and copy the login command from the OpenShift console user menu to set up your Kubernetes context.</p>

<h2>Set up Prometheus</h2>

<p>To get Prometheus working with OpenShift Streams for Apache Kafka, use the examples in the Prometheus documentation to create an <a href="https://github.com/prometheus-operator/prometheus-operator/tree/main/example/additional-scrape-configs">additional scrape config</a>. You will need to make a couple of modifications to your configuration.</p>

<h3>Create an additional config for Prometheus</h3>

<p>First, create the additional config file for Prometheus. To do this, create a file called <code>prometheus-additional.yaml</code> with the following content:</p>

<pre>
<code>- job_name: "kafka-federate"
  static_configs:
  - targets: ["api.openshift.com"]
  scheme: "https"
  metrics_path: "/api/kafkas_mgmt/v1/kafkas/<Your Kafka ID>/metrics/federate"
  oauth2:
    client_id: "<Your Service Account Client ID>"
    client_secret: "Your Service Account Client Secret"
    token_url: "https://identity.api.openshift.com/auth/realms/rhoas/protocol/openid-connect/token</code></pre>

<p>The angle brackets (<code><></code>) in the listing indicate details you'll need to gather from your own OpenShift Streams for Apache Kafka environment:</p>

<ul><li>You can see your Kafka ID by clicking on your <strong>Kafka Instance</strong> in the OpenShift Streams for Apache Kafka console.</li>
	<li>Follow OpenShift Streams for Apache Kafka <a href="https://access.redhat.com/documentation/en-us/red_hat_openshift_streams_for_apache_kafka/1/guide/f351c4bd-9840-42ef-bcf2-b0c9be4ee30a">getting started guide</a> to create a Kafka instance and service account for each instance. As described in the guide, copy and paste the client ID and client secret into <code>prometheus-additional.yaml</code>.</li>
</ul><h3>Create a Kubernetes secret</h3>

<p>Now, you need to create a Kubernetes secret that contains this config file:</p>

<pre>
<code>kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml --dry-run -o yaml | kubectl apply -f - -n <namespace></code></pre>

<h3>Apply your changes</h3>

<p>Finally, apply <code>prometheus.yaml</code>, <code>prometheus-cluster-role-binding.yaml</code>, <code>prometheus-cluster-role.yaml</code>, and <code>prometheus-service-account.yaml</code> using <code>kubectl apply -f <filename></code>.</p>

<h2>Set up Grafana</h2>

<p>To get Grafana working, use the <a href="https://github.com/grafana-operator/grafana-operator/blob/master/deploy/examples/GrafanaWithIngressHost.yaml">GrafanaWithIngressHost.yaml</a> example code from the Grafana project's GitHub repository. Remove the <code>hostname</code> field from the file, as OpenShift Dedicated will assign the hostname automatically.</p>

<p>Now, find the URL for Grafana from the OpenShift console <strong>Routes</strong> section, and open Grafana. The login details for Grafana are in the Grafana custom resource.</p>

<p>Next, connect Grafana to Prometheus by navigating to <strong>Settings -> Data Sources</strong>. Click <strong>Add data source</strong>, then click <strong>Prometheus</strong>. At that point, all you need to do is enter <code>http://prometheus-operated:9090</code>, the URL of the service, then click <strong>Save & Test</strong>.</p>

<p>You should now find metrics for your Kafka cluster in Grafana. Figure 2 shows a Grafana dashboard that displays most of the metrics available with OpenShift Streams for Apache Kafka.</p>

<div class="rhd-c-figure">
  <article class="media media--type-image media--view-mode-article-content"><div class="field field--name-image field--type-image field--label-hidden field__items">
              <a href="https://developers.redhat.com/sites/default/files/prometheus-fig2.jpg" data-featherlight="image"><img src="https://developers.redhat.com/sites/default/files/styles/article_floated/public/prometheus-fig2.jpg?itok=dF75OCFB" width="600" height="516" alt="A Grafana dashboard for Red Hat OpenShift Streams for Apache Kafka." typeof="Image" /></a>

          </div>
    <div class="field field--name-field-caption field--type-string field--label-hidden field__items">
                <div class="rhd-c-caption field__item">
        Figure 2. A sample Grafana dashboard for OpenShift Streams for Apache Kafka.
            </div>
          </div>

      </article></div>
<p>The JSON that defines this dashboard is <a href="https://github.com/pmuir/rhosak-grafana-dashboard">available on GitHub</a>.</p>

<h2>Conclusion</h2>

<p>In this article, I've shown you how to use Prometheus and Grafana to observe an OpenShift Streams for Apache Kafka instance. Prometheus is a very widely adopted format for monitoring and can be used with almost all observability services and products.</p>
The post <a href="https://developers.redhat.com/articles/2021/12/17/how-connect-prometheus-openshift-streams-apache-kafka" title="How to connect Prometheus to OpenShift Streams for Apache Kafka">How to connect Prometheus to OpenShift Streams for Apache Kafka</a> appeared first on <a href="https://developers.redhat.com/blog" title="Red Hat Developer">Red Hat Developer</a>.
<br /><br />
</description>
<pubDate>Thu, 13 Jan 2022 07:00:00 +0000</pubDate>
<dc:creator>Bob Reselman</dc:creator>
<guid isPermaLink="false">19d5abe2-3efb-49c1-91ad-8752f9686426</guid>
</item>
</channel>
</rss>
