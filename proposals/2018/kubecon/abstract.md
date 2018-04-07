# Abstract
When we redesigned CircleCI from the ground-up in 2016 and 2017, we knew we needed a proper scheduling solution to run jobs efficiently across our resources. Proper isolation of CircleCI's infrastructure and individual customer builds was one of the core requirements of this undertaking: for security, scalability, and reliability, we had to make sure to keep our system separate from our users' jobs. In this talk, I'll share why we chose Nomad and Kubernetes to solve these issues over other tools, present the issues and obstacles we solved, and talk about how this setup is running for us in production ~10 months after GA.

TODO: Find archive of rest of submission.
