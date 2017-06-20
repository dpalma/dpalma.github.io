---
layout: post
title: Making Better Promises
category: NodeJS
comments: yes
---

Promises can be difficult to work with at first. It is easy to go down a wrong path and end up with incorrect or unmaintainable code. This post discusses how I started using Promises in a live project. It focuses on a RESTful GET handler, the problems in the code as it was first written, and how it was improved later.
<!--more-->

Shown below is the GET handler for a RESTful controller that serves up a single individual listing. Multiple listings are handled in a separate controller. A listing posted by a given user will have that user&apos;s ID in its &quot;postedBy&quot; field. A listing that originates from the system or some other source will have no &quot;postedBy&quot; field. Conditional logic like this can be tricky to implement with Promises and that&apos;s how things started to go wrong for this code.

The requirements for this handler could be summarized like this:

- Take a listing ID argument as a URL parameter
- Report a bad request error if the ID is improperly formatted
- If the listing is not found then report that error
- Look up the user identified by the postedBy field if necessary
- Last, respond with a listing that includes conact info, if present

The code below shows the first attempt to implement a handler that satisfies these requirements.

<figure>
<figcaption>
Listing 1: First try
</figcaption>
{% highlight js linenos %}
ListingController.prototype.get = function(req, res) {
  let models = this.models, logger = this.logger;
  try {
    var id = new mongoose.Types.ObjectId(req.params.listingid);
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
</figure>

There are a couple major problems with this code.

First, it uses both a regular Javascript try-catch block, and a Promise-style catch handler. Having two separate error-handlers is a bad smell. Plus, the Promise catch block is catching indescriminate errors and returning a generic 500 error code. It is not clear what this is even handling.

Second, it creates a new Promise within the handler of a previous Promise, which causes [this warning](http://bluebirdjs.com/docs/warning-explanations.html#warning-a-promise-was-created-in-a-handler-but-was-not-returned-from-it). In this situation, the outer Promise handler chain will not wait for the inner one. This causes no ill effects in the code as written because the inner Promise results in either an error, or the response being sent. However, if the logic of the handler changes in the future it could cause hard-to-find bugs.

The new code below shows the first major round of improvements. Error handling has been unified into one catch block. The new code starts with a Promise.try() call instead of a regular try-catch. This sends any casting errors that result from invalid IDs to the single catch block at the end. The outer and inner Promises were combined into a single chain. Each then block is a single step in the chain. The steps are:

1. Convert the ID argument to a Mongoose database ID (produces a listing ID)
2. Look up the listing in the database (produces a listing object)
3. Look up the postedBy user ID if present (produces a user object or null)
4. Get the preferred contact info from the user and send the response

The key to this change was to think of each Promise as a single step in the process. Each then-block receives the result of the previous step as its input. For the conditional case, where a listing may or may not have been posted by a user, the user is simply null when there is no postedBy field. If there is a postedBy field, a database query Promise leads to the next step. Otherwise, a call to Promise.resolve(null) indicates to the next step that there is no user from which to get contact information.

<figure>
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
</figure>

Is Listing 2 still not good enough? Have thoughts for further improvement? Let me know in the comments!