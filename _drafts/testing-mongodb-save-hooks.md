---
layout: post
title: Testing MongoDB save hooks
category: nodejs
comments: yes
---

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
