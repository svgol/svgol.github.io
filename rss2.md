---
layout: none
title: "The example of an rss item with an empty attribute"
permalink: /rss2/
---

<rss version="2.0"
     xml:base="https://developers.redhat.com/"
     xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <atom:link href="https://svgol.github.io/rss2" rel="self" type="application/rss+xml"/>
    <title>svgol' another test feed</title>
    <!-- <link>https://https://svgol.github.io/rss2</link> -->
    <description>
      Stories and tutorials on the latest technologies in cloud application development.
    </description>
    <language>en</language>
    <lastBuildDate>Thu, 20 Jan 2022 07:00:00 +0000</lastBuildDate>
    <sy:updatePeriod>hourly</sy:updatePeriod>
    <sy:updateFrequency>1</sy:updateFrequency>
    <item>
      <title>Reduce the size of container images with DockerSlim</title>
      <link>https://developers.redhat.com/articles/2022/01/17/reduce-size-container-images-dockerslim</link>
      <description>

<p><a href="">Containers</a><a href="https://developers.redhat.com/topics/containers"> </a>are a great way to package your applications. Packaging your application codebase together with its dependencies creates a <em>container image</em>. The smaller the container image is, the faster your application will spin up for the first time, and the faster it will scale. But many container images are quite large, in the hundreds of megabytes—just search Docker Hub and prepare to be amazed at the image sizes.</p>

<p>In this article, you'll learn how to optimize Docker container images for size using a project called <a href="https://github.com/docker-slim/docker-slim">DockerSlim</a>. DockerSlim, which is open sourced under the Apache 2.0 license, won't change anything in your container image, but can still reduce its size—or <em>minify</em> it—by up to a factor of 30. For applications written in compiled languages, the size reduction can be even more dramatic. DockerSlim also makes your packages more secure by reducing the available attack surface.</p>

<h2>Using DockerSlim</h2>

<p>DockerSlim uses various techniques to optimize and secure container images. For one, it disposes of packages and files from the container image that your application does not need to serve the business logic. Removing additional files helps to reduce the container's attack surface. Using DockerSlim is a simple two-step process.</p>

<h3>Step 1: Install DockerSlim</h3>

<p>The installation steps for DockerSlim are explained in the <a href="https://github.com/docker-slim/docker-slim#downloads">DockerSlim GitHub repository</a>. Here's how you would install DockerSlim on macOS using <code>brew</code>:</p>

<pre>
<code class="language-bash">brew install docker-slim
</code></pre>

<h3>Step 2: Minify your Docker image</h3>

<p>Once you have DockerSlim installed, you can use it to minify your Docker image by running <code>docker-slim</code> against existing container images. Start by running <code>docker images</code> to list the container images currently available:</p>

<pre>
<code class="language-bash">$ docker images
REPOSITORY                    TAG            IMAGE ID       CREATED         SIZE
python-hello-world-ubi       latest          50c12e1ca549   13 days ago     169MB</code></pre>

<p>Next, run <code>docker-slim build</code> against an existing container image to minify it:</p>

<pre>
<code class="language-bash">$ docker-slim build python-hello-world-ubi
docker-slim: message='join the Gitter channel to ask questions or to share your feedback' info='https://gitter.im/docker-slim/community'
docker-slim: message='join the Discord server to ask questions or to share your feedback' info='https://discord.gg/9tDyxYS'
docker-slim: message='Github discussions' info='https://github.com/docker-slim/docker-slim/discussions'
cmd=build info=param.http.probe message='using default probe'
cmd=build state=started
cmd=build info=params rt.as.user='true' keep.perms='true' tags='python-hello-world-ubi.slim ' target.type='image' target='python-hello-world-ubi' continue.mode='probe'
cmd=build state=image.inspection.start
...
...
...
cmd=build state=done
cmd=build info=commands message='use the xray command to learn more about the optimize image'
...
...
</code></pre>

<p><code>docker-slim</code> will generate an optimized image from your source image and store that with a <code>.slim</code> file extension:</p>

<pre>
<code class="language-bash">$ docker images
REPOSITORY                        TAG        IMAGE ID       CREATED         SIZE
python-hello-world-ubi            latest     50c12e1ca549   13 days ago     169MB
python-hello-world-ubi.slim       latest     17ba0fab2e4e   13 days ago     25.8MB
</code></pre>

<p>In this case, the original image is 169MB, and the newly optimized image is 25.8MB—a <em>600 percent</em> reduction in size.</p>

<h2>Get a DockerSlim container report</h2>

<p>If you're curious, you can use the <code>docker-slim xray</code> command to get the details about a package's size. The command performs a static analysis on the target container image and reverse-engineers the Dockerfile from the image, telling you what's inside of your container image and why it is so big:</p>

<pre>
<code class="language-bash">$ docker-slim xray python-hello-world-ubi</code></pre>

<p>The <code>xray</code> option on <code>docker-slim</code> generates a <code>slim.report.json</code> file that includes all the details on your container image and provides a great resource for investigating what's inside of it. Here is partial output from an example file:</p>

<pre>
<code class="language-bash">...
...
cmd=xray info=image id='sha256:50c12e1ca549655293c8148359bf80d551cec739c34556d4506f49602f9ea203' size.bytes='168702969' size.human='169 MB'
cmd=xray info=image.stack index='0' name='python-hello-world-ubi:latest' id='sha256:50c12e1ca549655293c8148359bf80d551cec739c34556d4506f49602f9ea203' instructions='8' message='see report file for details'
...
...
cmd=xray info=layer.objects.top.start
A: mode=-rw-r--r-- size.human='6.9 MB' size.bytes=6949250 uid=0 gid=0 mtime='2021-03-05T16:50:01Z' H=[A:0] '/usr/lib/locale/C.utf8/LC_COLLATE'
A: mode=-rw-r--r-- size.human='5.2 MB' size.bytes=5194744 uid=0 gid=0 mtime='2021-01-29T16:23:30Z' H=[A:0] '/usr/share/misc/magic.mgc'
A: mode=-rw-r--r-- size.human='4.8 MB' size.bytes=4837376 uid=0 gid=0 mtime='2021-09-14T16:20:29Z' H=[A:0/M:2] '/var/lib/rpm/Packages'
A: mode=-rwxr-xr-x size.human='3.2 MB' size.bytes=3167976 uid=0 gid=0 mtime='2021-03-05T17:03:03Z' H=[A:0] '/usr/lib64/libc-2.28.so'
A: mode=-rwxr-xr-x size.human='3.1 MB' size.bytes=3071456 uid=0 gid=0 mtime='2021-03-25T16:49:50Z' H=[A:0] '/usr/lib64/libcrypto.so.1.1.1g'
A: mode=-rwxr-xr-x size.human='2.2 MB' size.bytes=2191840 uid=0 gid=0 mtime='2021-03-05T17:03:04Z' H=[A:0] '/usr/lib64/libm-2.28.so'
A: mode=-rwxr-xr-x size.human='2.1 MB' size.bytes=2052344 uid=0 gid=0 mtime='2021-04-01T13:15:58Z' H=[A:0] '/usr/lib64/libgnutls.so.30.28.0'
A: mode=-rwxr-xr-x size.human='2.0 MB' size.bytes=1975976 uid=0 gid=0 mtime='2021-03-08T15:43:28Z' H=[A:0] '/usr/lib64/libdnf.so.2'
A: mode=-rwxr-xr-x size.human='1.9 MB' size.bytes=1869272 uid=0 gid=0 mtime='2021-09-09T06:45:53Z' H=[A:0] '/usr/lib64/libdb-5.3.so'
A: mode=-rwxr-xr-x size.human='1.8 MB' size.bytes=1770160 uid=0 gid=0 mtime='2021-07-26T13:46:03Z' H=[A:0] '/usr/lib64/libgio-2.0.so.0.5600.4'
A: mode=-rwxr-xr-x size.human='1.8 MB' size.bytes=1760264 uid=0 gid=0 mtime='2018-08-17T20:47:18Z' H=[A:0] '/usr/lib64/libunistring.so.2.1.0'
A: mode=-rwxr-xr-x size.human='1.7 MB' size.bytes=1661376 uid=0 gid=0 mtime='2020-09-29T22:13:10Z' H=[A:0] '/usr/lib64/libstdc++.so.6.0.25'
A: mode=-rwxr-xr-x size.human='1.5 MB' size.bytes=1503528 uid=0 gid=0 mtime='2021-05-19T08:14:20Z' H=[A:0] '/usr/lib64/libxml2.so.2.9.7'
A: mode=-rwxr-xr-x size.human='1.5 MB' size.bytes=1503456 uid=0 gid=0 mtime='2019-06-14T08:41:36Z' H=[A:0] '/usr/lib64/libgmp.so.10.3.2'
A: mode=-rwxr-xr-x size.human='1.4 MB' size.bytes=1367288 uid=0 gid=0 mtime='2021-07-28T11:56:34Z' H=[A:0] '/usr/lib64/libsystemd.so.0.23.0'
A: mode=-rwxr-xr-x size.human='1.3 MB' size.bytes=1321808 uid=0 gid=0 mtime='2021-03-05T17:03:06Z' H=[A:0] '/usr/sbin/ldconfig'
A: mode=-rwxr-xr-x size.human='1.3 MB' size.bytes=1304120 uid=0 gid=0 mtime='2020-04-14T12:13:07Z' H=[A:0] '/usr/bin/coreutils'
A: mode=-rwxr-xr-x size.human='1.2 MB' size.bytes=1246520 uid=0 gid=0 mtime='2021-01-11T13:56:24Z' H=[A:0] '/usr/lib64/libp11-kit.so.0.3.0'
A: mode=-rwxr-xr-x size.human='1.2 MB' size.bytes=1188080 uid=0 gid=0 mtime='2020-06-15T16:21:33Z' H=[A:0] '/usr/lib64/libgcrypt.so.20.2.5'
A: mode=-rwxr-xr-x size.human='1.2 MB' size.bytes=1167784 uid=0 gid=0 mtime='2021-07-26T13:46:03Z' H=[A:0] '/usr/lib64/libglib-2.0.so.0.5600.4'
cmd=xray info=layer.objects.top.end
cmd=xray info=layer.end
...
...</code></pre>

<h2>Conclusion</h2>

<p>Bloated container images can negatively impact application performance. If you suspect your container image is getting too big, after reading this article you should know how to use DockerSlim to minify it. It's a nice addition to your toolbox.</p>
The post <a href="https://developers.redhat.com/articles/2022/01/17/reduce-size-container-images-dockerslim" title="Reduce the size of container images with DockerSlim">Reduce the size of container images with DockerSlim</a> appeared first on <a href="https://developers.redhat.com/blog" title="Red Hat Developer">Red Hat Developer</a>.
<br /><br />
</description>
<pubDate>Mon, 17 Jan 2022 11:00:00 +04</pubDate>
<dc:creator>Karan Singh</dc:creator>
<guid isPermaLink="false">19d5abe2-3efb-49c1-91ad-8752f9686426</guid>
</item>
</channel>
</rss>
