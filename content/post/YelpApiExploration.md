---
date: 2017-10-08T21:16:33+02:00
draft: false
title: Yelp Api Exploration
tags:
- code
type: post
---

The basic motivation for this project is to have an App that can search for yelp businesses, select a business, and provide some *neat* information on that business (something the standard app might not provide). This is largely a toy application to test some new things out. 

This project was an experiment with a number of different frameworks such as: 
- ReactJS + Typescript frontend 
- Websockets communication between back and front ends
- Asp.Net core backend 
- Yelp API 



## Yelp API 

The queries to the yelp API are extremely simple; the client builds up a query string for each endpoint that it hits, sends the query, and parses the JSON back to c#. More to be done here-- 

## Designing the Frontend 

The frontend was very simple & created to experiment with react. The basic functionality is as follows: 
- The user types into the search bar, this sends a new query to the websocket server
- The server replies with possible results
- The Frontend displays these results underneath the query (like autocomplete)
- The user clicks a search result, sending a message over websockets requesting that page / result 
- The server replies with the new page, which is rendered 

All of this was written in typescript. Typescript with react also worked as expected, the only problem came with working with Protobuf(js). 

## Websockets & Protobuf

To keep track of the messages sent to and from the server, I used google [protobuf](https://developers.google.com/protocol-buffers/). The message format is defined in a special language-agnostic file. Then, the protobuf compiler compiles protobuf into real classes for whatever language you are using. These classes basically have just two function, serialize to and deserialize from some format (json or binary). This takes most of the hassle out of serializing and unpacking objects between different languages. 

Originally, I hoped to send protobuf binary buffers over the tcp socket, and then have protobuf serialization & deserializations take care of everything. But, protobuf as binary did not play nicely with typescript & javascript; deserializing a binary buffer usually just threw an error and failed to work. Once I got it working, the types were incorrect as protobuf js did not seem to have great typescript support. So, I instead used protobufs JSON desrialization. These JSON messages made up all of the communication between the client and server. 

The frontend used a singleton class to manage all the server-client communication. Different components could 'subscribe' to message types by adding a function handler to a message type. 