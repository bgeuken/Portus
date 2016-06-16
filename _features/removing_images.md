---
layout: post
title: Removing images and tags
order: 5
longtitle: Removing images and tags from the registry through Portus
---

<div class="alert alert-info">
  Only available in <strong>2.1 or later</strong>.
</div>

## Intro

One of the most remarkable features from the 2.1 release is the ability to
remove images and tags. This has not been possible until now because, even
though it has always been possible to delete an image by manifest (*soft
delete*), the orphaned blobs couldn't be properly removed. Therefore, we
decided that it was better to not provide this feature at all because otherwise
we would be lying to our users (by saying that an image was deleted when in
fact it wasn't really).

Fortunately, Docker Distribution 2.4 implemented a Garbage Collector, which
basically removes all the blobs that have been orphaned (no manifest is
referencing them). We have two challenges though:

1. Maintenance has to be done manually (see [Maintenance](#maintenance)).
2. Portus has no way to check the version of the running registry, so the
   administrator has to
   [explicitly enable this](/docs/Configuring-Portus.html#delete-support).

## Removing images & tags

Removing images and tags is quite intuitive from a user point of view. Just go
to the repository page:

![Repository page](/build/images/docs/repo-images.png)

In there, you can see trash cans. Click on any of them and a confirmation form
will pop up. If you agree, then you will have removed the specified tag or
image. Moreover, all this is still tracked in the activities list. For
example, let's say that in the previous example we removed the `latest` tag.
Then we would have the following activities list:

![Activities of removing a tag](/build/images/docs/repo-activities1.png)

Now, in the same repository page, if we want to remove the image and all its
tags, we can just simply click on the "Delete image" link located on the top
right corner of the page. In the previous example, if now we removed the whole
image, we would get the following activities list:

![Activities of removing an image](/build/images/docs/repo-activities2.png)

## Maintenance

The Garbage Collector (GC from now on) is a crucial part on this feature and
it's implemented by Docker Distribution. The bad thing is that it has been
provided as a separate command. This means that:

1. Administrators have to call this command explicitly, instead of it being
   handled automatically for us.
2. In order to do it safely in production some downtime is to be expected. You
   can run the GC anytime you want, but it will bring concurrency issues if
   executed when some pushes were being performed. In order to avoid that, you
   should thus stop the registry, run the GC and start the registry again.

Despite this inconvenience in maintenance, running the GC is actually quite
simple. You just have to call the registry command with the new
`garbage-collect` command. It accepts one argument: the configuration file.
Moreover, if you just want to check whether there are orphaned blobs or not,
you can pass it the `-d` flag.
