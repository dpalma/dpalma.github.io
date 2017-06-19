---
layout: post
title: Making Better Promises
category: NodeJS
comments: yes
---

Promises can be difficult to work with at first. It is easy to go down a wrong path and end up with incorrect or unmaintainable code. This post discusses how I started using Promises in a live project. It focuses on a RESTful GET handler, the problems in the code as it was first written, and how it was improved later.
<!--more-->

Shown below is the GET handler for a RESTful controller that serves up a single individual listing. Multiple listings are handled in a separate controller. A listing posted by a given user will have that user&apos;s ID in its &quot;postedBy&quot; field. A listing that originates from the system or some other source will have no &quot;postedBy&quot; field. Conditional logic like this can be tricky to implement with Promises and that&apos;s how things started to go wrong for this code.

First, it uses both a regular Javascript try-catch block, and a Promise-style catch handler.

Second, it creates a new Promise within the handler of a previous Promise, which causes [this warning](http://bluebirdjs.com/docs/warning-explanations.html#warning-a-promise-was-created-in-a-handler-but-was-not-returned-from-it).

Listing 1: First try
{% highlight js linenos %}
ListingController.prototype.get = function(req, res) {
  let models = this.models, logger = this.logger;
  try {
    var id = new mongoose.Types.ObjectId(req.params.gameid);
    var result;
    models.Listing.findById(id).then(function(listing) {
      if (listing) {
        result = listing.toObject();
        if (result.postedBy) {
          models.User.findById(result.postedBy).then(function(user) {
            var contact = user.getPreferredContactInfo();
            if (typeof(contact) !== 'undefined') {
              result.contact = contact;
            }
            res.send(result);
          })
        } else {
          res.send(result);
        }
      } else {
        logger.info("Listing entry not found", id);
        res.sendStatus(404);
      }
    }).catch(function(err) {
      logger.error(err);
      res.sendStatus(500);
    });
  } catch (err) {
    // invalid Mongoose object identifier
    logger.error(err);
    res.sendStatus(400);
  }
}
{% endhighlight %}

Listing 2: Some improvement
{% highlight js linenos %}
ListingController.prototype.get = function(req, res) {
  let models = this.models, logger = this.logger, theListing;
  Promise.try(function() {
    let id = new mongoose.Types.ObjectId(req.params.listingid);
    return Promise.resolve(id)
  }).then(function(listingId) {
    return models.Listing.findById(listingId)
  }).then(function(listing) {
    if (listing) {
      theListing = listing;
      if (listing.postedBy) {
        return models.User.findById(listing.postedBy)
      } else {
        return Promise.resolve(null);
      }
    } else {
      return Promise.reject(404);
    }
  }).then(function(user) {
    let result = Object.assign(theListing.toObject(), {
      contact: user && user.getPreferredContactInfo()
    })
    res.send(result);
  }).catch(function(err) {
    if (typeof err === "number") {
      res.sendStatus(err);
    } else {
      // invalid Mongoose object identifier
      logger.error(err);
      res.sendStatus(400);
    }
  })
}
{% endhighlight %}
