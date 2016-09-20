---
layout: post
title: Unit-testing MongoDB models with save hooks
category: NodeJS
comments: yes
---

Data model classes with address fields may well use a geocoder, such as the Google Maps API, to get the latitude/longitude coordinates of a given address. These coordinates are vital to applications like store locators. In a Mongo model, a save hook can keep the coordinates in sync with any changes to the address fields. This post will demonstrate one way to compose the schema, model, and test so that the save hook can be mocked during unit tests.
<!-- more -->

Consider a test like the following:

```javascript
describe("StoreLocation model", function() {
  it("fills in the geo location fields", function(done) {
    StoreLocation.create({
      date: faker.date.future(),
      notes: faker.lorem.words(),
      address: {
        name: "Some Place",
        street: "1 Main St",
        city: "Boston",
        state: "MA"
      }
    }).then((storeLoc) => {
      expect(storeLoc.location.type).to.equal("Point");
      expect(storeLoc.location.coordinates[0]).to.equal(-71.011754);
      expect(storeLoc.location.coordinates[1]).to.equal(42.381862);
      storeLoc.remove().then(function() { done() });
    });
  });
});
```

Assuming 


# Unit-Testing Mongo Models with Save Hooks

## Problem
* Models with location fields use a geo-coding API to retrieve coordinates
* Calling Google Maps or another geocoder during a unit test is unacceptable
* Mongo models can make it tricky to overwrite or inject a mock geocoder
* Separating model from schema makes it possible mock the geocoder to enable testing the model

## Solution
* Separate schema into its own file
* In unit test, load the schema with rewire
* Override the geocoder with a mock
* Create a separate Mongo model for testing using the re-wired schema
