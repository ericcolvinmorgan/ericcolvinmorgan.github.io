---
layout: post
title:  "Calculation Engine"
date:   2021-05-09 10:42:33 -0700
categories: projects
tags: c++ c# excel calcengine
---
One of the larger complaints encountered by tax modeling teams and clients/auditors consuming models is a lack of transparency into calculations produced by SaaS systems.  Coupled with the fact that regulations can change frequently and/or legal positions can differ dramatically depending on various fact patterns models/calculations need to be updated very often leading to a proliferation of ad-hoc Excel models tweaked in various ways.  While new positions can be added to a development roadmap, turn around time to develop and move through the various change management layers is often unacceptable when deadlines are measured in days and not weeks.  This project was created in an attempt to address some of these concerns.

Excel is powerful in the sense that it is very easy to use and produce auditable reports, though managing and updating 100s of files with new positions is tedious.  The CalcEngine includes a C#-based service that tokenizes and extracts the calculation tree from a base file produced by subject matter experts.  Once a dataset has been produced, an additional C++-based calculation service runs CSV datasets through the calculation trees producing calculations based on the Excel logic provided by SMEs.  The service does not have dependencies on Excel, so can easily be deployed where needed (an EC2 instance, AWS Lambda, etc.).  This allows users to quickly update a theoretical SaaS system with new calculations, while maintaining the calculation tree enabling the production of auditable reports downstream.  

[Calculation Engine](https://github.com/ericcolvinmorgan/CalcEngine)