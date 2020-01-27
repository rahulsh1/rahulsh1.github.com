---
layout: post
title: AWS Certified Solution Architect Associate Exam
category: tech
tags: AWS Certified Solution Architect Associate
year: 2020
month: 01
day: 09
published: true
summary: Exam experience, resources and thoughts
---

The [AWS Certified Solutions Architect - Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/) examination is intended for individuals who perform a solutions architect role and have one or more years of hands-on experience designing available, cost-efficient, fault-tolerant, and scalable distributed systems on AWS.

Note that a new version of the AWS Certified Solutions Architect - Associate exam will be available in March 2020 (SAA-C02).

> Exam Format
- The exam has 65 questions
- Multiple choice with single answer as well as two answer questions
- Passing score is 700/1000 (70%)
- No penalty of guessing
- 130 minutes to complete the exam
- You can mark questions for later review

### Topics for the exam and its weightage
- Design Resilient Architectures - (34%)
- Define Performant Architectures - (24%)
- Secure Applications and Architectures - (26%)
- Design Cost-Optimized Architectures - (10%)
- Define Operationally Excellent Architectures - (6%)

### Exam Study Resources

#### PluralSight
I started with PluralSight Course [AWS Certified Solutions Architect - Associate](https://app.pluralsight.com/paths/certificate/aws-certified-solutions-architect-associate). This is 20+ hours of videos. You can watch these at 1.2x speed.
While they cover a good set of AWS services and labs, they do not cover in sufficient depth that is needed for the exam.
The last course "Demystifying the AWS Certified Solutions Architect: Associate Exam" does focus on the exam topics at a high level and how to prepare.

#### Udemy
To gather some more ground, I took following two Udemy courses:
1. [AWS Certified Solutions Architect - Associate 2020](https://www.udemy.com/course/aws-certified-solutions-architect-associate/) by  Ryan Kroonenburg.
  This is 13 hours and has 2 Practice Tests. While some of the topics are covered well, it lacks details on some of the more advanced topics like Redshift, CloudFront, AWS Interconnects.

2. [Ultimate AWS Certified Solutions Architect Associate 2020](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c01/) by Stephane Maarek.
  This is 18.5 hours and has 1 Practice test. This is by far the most extensive course and has good coverage overall with focus on the exam. There are these architecture discussion videos which can you help in understanding the big picture and how these services all come together.

#### QwikLabs
These are good for some hands-on lab to solidify your understanding and gain more experience with AWS services.

> The above material (if you know these very well) can get you close to 60-70% of what is needed for the exam.

#### AWS Resources
AWS has a ton of resources and it is easy to get lost in the documentation hole. From an exam point of view, the FAQs have a lot of info not covered anywhere else.

Start with the [FAQ Site](https://aws.amazon.com/faqs/). Some of these are really long and can take some time to go through.

Here are the FAQS that I went through:
- [EC2 FAQ](https://aws.amazon.com/ec2/faqs/)
- [S3 FAQ](https://aws.amazon.com/s3/faqs/)
- [RDS FAQ](https://aws.amazon.com/rds/faqs/)
- [VPC FAQ](https://aws.amazon.com/vpc/faqs/)
- [Route 53 FAQ]( https://aws.amazon.com/route53/faqs/)
- [SQS FAQ](https://aws.amazon.com/sqs/faqs/)
- [ELB FAQ](https://aws.amazon.com/elb/faqs/)
- [DynamoDB FAQs](https://aws.amazon.com/dynamodb/faqs/)
- [Redshift FAQs](https://aws.amazon.com/redshift/faqs/)

Also there are these additional resources - some I read the entire paper and some I just skimmed through.
- [AWS Whitepapers](http://aws.amazon.com/whitepapers)
- [AWS Well Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Case Studies](https://aws.amazon.com/solutions/case-studies/)
- [AWS Documentation](https://docs.aws.amazon.com/index.html)

### Practice Exams

AWS has a exam readiness course. You can take this even before you begin with the course material. That can give you some insight on how the questions will be.
See [Exam Readiness: AWS Certified Solutions Architect – Associate (Digital)](https://www.aws.training/Details/Curriculum?id=20685)

Also this [Practice exam](https://www.udemy.com/course/aws-certified-solutions-architect-associate-amazon-practice-exams/) from Udemy by Jon Bonso and Tutorials Dojo is the one I took after reading a bunch of reviews. I learned a lot from the explanations and the additional links given in the answers.

Definitely take the practice exam offered by AWS as this will help you get familiar with the UI for the final exam and the pattern for the questions.

### Exam Tips
Some tips that may help you crack the exam:
- Read the Questions carefully for what is being asked. Some focus on scalability, availablity, cost and some others.
- Read all the answers even if you think you got the right one
- Eliminate the ones which are wrong answers first
- If unsure, pick answers from what you know
- Pick a simple solution rather than something complex

### Summary
While the exam focuses on your ability to pick the correct solution as an architect, you want to think about the following things as you prepare and take notes from the various learning materials.
- What is this service good at? What is the use-case where this will be used?
- What are the alternatives if any?
- What is the availability? How to make it highly available?
- How can you scale this?
- How much does it cost?
- Are there other cost-effective options?
- Is there a newer simpler solution to handle this?

You should be able to understand all the AWS services covered and be able to reason why one solution is good over another.

Till then... :metal:
