---
layout: post
title: "Flatiron Connection"
permalink: "flatiron-connection"
date: 2015-04-12 12:53:00 -0500
categories: web-development, projects
---
This is a project I started back in week 4 or 5 of the semester, but never got around to finishing it. I started the app on Sinatra, but in 1 1/2 days, I converted the app to Rails, added some more functionality and got it up on Heroku.

The basic feature of this app is that in the seed file, I use a scraper class to scrape some student information from the Flatiron Students website that every cohort creates. Once a user authenticates using Twitter, the user can see a student roster for each cohort. From the roster page, you can choose to add new students in case the scraper missed some or follow all the students on Twitter. If you click on the individual student link, it will take you to their condensed profile page. There's some cool css effects on the panels so make sure to check it out!

Here's a link to the app: <a href="http://flatiron-connection.herokuapp.com/" target="_blank">http://flatiron-connection.herokuapp.com/</a>. Just a heads up, if you click "Follow Students", it will automatically follow all students shown on the page! If you don't want to follow all the students, don't click on it!

