---
author: "Jenni Lei"
categories: [Visual Analytics, R]
date: 2019-06-06
title: "Deploy Social Network App using Shiny"
featured: "social.png"
featuredpath: "date"
linktitle: "jfhkf"
type : "post"
---

This post will summarize our group final project for DSBA 5122: Visual Analytics (class website: https://dsba5122-spring2019.netlify.com/). Using a hypothetical scenario, we, as data scientists, developed a shinydashboard on high school social network with key centrality measures to identify the most popular peers in the circle. 

<b>Web link to shiny App:</b> [https://balavigneswaran-kuppusamy.shinyapps.io/network/]

<b>Visit Github Repository:</b> [https://github.com/jlei2821/network_visual_design]

<i>Special shoutout to my teammates Balavigneswaran Kuppusamy and Minglan Ye for all your time and efforts contributed to this project, which earned us the "Best Human-Centered Machine Learning App" in class. </i>

<br/>
## Introduction
A high school in Marseille, France has conducted an analysis on historical data studying the influence of different factors on academic performance. The study observes student behavior and rates each student’s peer connections on a scale of 1 to 10. Results show that receiving a high score on peer connections is highly correlated with not only academic excellence of the student while in school, but also superior development after graduation.

In order to enhance frequent communication among the student body, the high school principal decides to employ data scientists to create an online social platform. The purpose of this platform is to create a convenient environment for students to quickly navigate through connections between fellow peers and identify ways of contact. A successful platform not only needs to be quickly accessible to students, but also encourages academic discussions among student groups.

The proposed platform should provide easy options for students to explore the community, & observe network strength and enable them to make contacts with the right students to enable greater level of connectivity & communication among students.

<br/>
## Operation & Abstraction Design
### Datasets

The data scientist team compiled two datasets for the purpose of this project after a 5-day survey

<p class="tab"><b>Contacts</b> - entails contacts of the students of nine classes. Data records indicate the IDs of students having the contact and the duration of contact on a level of 1-4, with 1 being the shortest and 4 being the longest. A clip of dataset is copied below:</p>

<div style="width:image width px; font-size:80%; text-align:center;"><img src="/img/contacts.png" height="130" width="470"></div>

<p class="tab"><b>Metadata</b> - contains student details like student ID, class ID, and gender. </p>
<div style="width:image width px; font-size:80%; text-align:center;"><img src="/img/metadata.png" height="130" width="470"></div>

<i>To generate network profile images, we used random user API with configurable parameters. This allows us to extract profile information like name, gender, ssn, profile pictures, etc.</i>

### Data Preparation Steps 
The following data preparation steps were executed from the 3 datasets to produce the nodes & edges needed for visual design:

1. Random user profiles loaded from web API, & converted from JSON format to a dataframe containing the variables of interest - e.g. name, gender, ssn (unique ID), profile picture, etc.
2. Data from Metadata dataset augmented with details from random user profiles (random mappings made between student metadata & user profiles). During the merge, care was taken such that students are matched with a user profile having the same gender. This data will be used as the nodes/vertices of the graph.
3. Data from Contacts dataset used as edges of the graph. The contact strenth considered as weight of the edge.
4. Graph built from the nodes & edges data using the igraph library.
igraph library provides various centrality measures for the network & can easily identify other network characteristics such as distance, shortest path, neighbors, etc.
5. Various centrality measures such as degree, centrality, closeness, betweenness, etc are calculated from the igraph.
6. The absolute values of the centrality measures are not in any way effective or usable by the students directly. Those absolute values have been transformed into Quartile measures between 1 - 4, where 4 indicating a student being in the highest quartile of the measure, & 1 the lowest.
7. These quartile measures are stored in the nodes dataframe against each node, & will be usable for multiple calculations to rank the students, etc.

<br/>
## Visual Encoding / Interaction Design

Students should be able to quickly navigate through peer networks displayed as graph / network map. Students will input into the application by logging into their profile, and a network map will be generated based on student information and strength of their relations indicated in the contacts data.

### Considerations in the Visual Encoding:
* Network map constructed using vizNetwork library using the nodes & edges, & is created as a force-directed graph layout.
* Each node in the network map will display student profile picture (thumbnail size).
* All the nodes will be of the same size, except the currently ‘logged in’ user, which will be shown in bigger size.
* The node will also indicate the class student belongs to in the form of differentiating color bands encircling the profile pictures.
* The edge thickness will depend on the strength of their contact & will be thicker if the contact is bidirectional.
* In a seperate panel (sidebar on the right), the student can see a detailed profiles information of him/herself, with the different centrality measures highlighted along with their quartile ranking in appropriate color.
* We filter through the network to make sure that most relevant people are properly displayed in friend suggestions (calculated based on centrality measures). For each profile, we will display top five friend suggestions, out of a pool of neighboring peers within the third and fourth quartiles of all centrality measures. 

### Considerations in the Interaction Design:
* Each node, when hovered over by mouse, will display the larger profile picture of the student, name, class & their student ID.
* User will be presented with a list of all users in the network & be able to switch to different student profile (mimicking login by different user). This ‘Log-in as’ will serve as a way for admins to monitor student activities and interactions.
* User will be able to toggle the network view where the size of the nodes will be based on their Degree centrality measure.
* User will be able to click on a node in the graph & view the detailed profile of the user & their centrality measures.
* User will be able to view the shortest path between him/herself & the selected node (in a text format). This specifies the easiest way to get to know that person, and who all can introduce the student to the party of interest.
* Based on the centrality measures of the selected user, the system will provide recommendations on whether the selected user can be considered for potential friendship.

To plot the network map and incorporate all intended features, igraph package was used to optimize the visual presentation of the graph and generate insights within the network using centrality functions. The following centrality measures are displayed as part of every student profile (self & the selected node):

* <b>Degree:</b> How many direct, ‘one hop’ connections each node has to other nodes within the network.
* <b>Eigen Centrality:</b> How well connected a node is, and how many links their connections have, and so on through the network.
* <b>Closeness:</b> Calculates shortest paths between all nodes, then assigns each node a score based on its sum of shortest paths. This helps to see the strength of relationship between an individual and others.
* <b>Betweenness:</b> Measures number of times a node lies on the shortest path between other nodes. Higher number indicates a person who is better connected in the high school circle.

<br/>
## Algorithmic Design

### Optimizations implemented for improving the overall modular design
* Shiny components were split into separate UI & server files to simplify the files.
* Various ‘non-shiny’ tasks like, data preparation, igraph building, etc have been grouped into separate source files (data.R, igraph.R) & embedded as a link from the server file. This reduces the amount of ‘noise’ in the shiny server & UI components.
* User profile is displayed for both the logged-in user & for selected node. Since it is common information, that component has been identified as a common component to be ‘modularized’. A seperate Shiny ‘module’ has been created for the UserProfile function & included via global.R.
* A separate shiny module has been created to display ‘Friend suggestions’ information to seggregate the functionality & make it reusable.

### Optimizations implemented for improving the application performance.

* The initial data preparation activity is very intense & time consuming. The data preparation steps & components are common for every user. So instead of rebuilding those data structures every time, the application automatically save those objets (nodes, edges, igraph) as ‘.rds’ files & just load them directly for subsequent uses.
visNetwork graph building is an expensive operation. Instead of rebuilding the whole network everytime for minor changes, visNetwork proves a ‘proxy’ through which specific modifications on the network in an optimized manner.
* Execute shiny load test & understand the application performance under higher load & identify performance bottlenecks, if any.

<i>Annnd we're done! Please leave me your thoughts or feedback in the contact section. We appreciate any suggestions on improving our app or algorithm! </i>

Love, <br/>
J