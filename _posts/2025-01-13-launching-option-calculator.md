---
layout: post
title: "Launching Option Calculator"
date: 2025-01-13
tags: [React, R, AWS, 2025-resolution, january]
---

Built [calculatemyoptions.click](https://calculatemyoptions.click/) website. It's entirely hosted on AWS. It's a SPA where static files are on S3 bucket and are served with CloudFront. Backend is R code hosted on Lambda. All of the infra is created/updated with AWS CDK.

In order to release this project, I had to figure out how to host R inside of a container and serve it with AWS Lambda. I've already done something similar in [Running R on AWS Lambda](/2024/07/26/running-r-on-aws-lambda.html), so I could re-use parts of learning from there and build on top of it.

## Challenges

There were a couple of challenges that I encountered when working on this project:

### R Libraries and Docker Image Size

Not all R libraries were available for AWS Lambda image, so I had to compile a couple of them from source code. When compiling, too many intermediate artifacts were created which put the final image over 10GB (Docker images hosted on AWS Lambda have a limit of 10GB [1]).

I reduced the size of the Lambda container by using multi-stage Docker build process and copying only compiled binaries into a final AWS Lambda image. I was able to go from 11GB to around 4GB, and I could run R container with all libs on AWS Lambda (yay).

### Frontend Development

The second challenge was the frontend since I've never done it before. Luckily ChatGPT helped me setup the React template that I could then modify and shape.

Also, CloudFront was a bit tricky to configure, specifically for configuring routes to Lambda function and making sure that the SPA could talk to Lambda and work across Firefox and Chrome.

## Testing and Release

After parts of the whole project had been configured, I did a couple of runs of integration testing and fixing. Once I checked that the skeleton and parts work together, I did a mini release on LinkedIn, to see what people say and if I can catch any errors with real traffic.

## Takeaways

Overall, it was a fun learning experience, and now I have deployment templates that I can leverage for future projects as well as knowledge about how website hosting on AWS is done.

### References

1. [AWS Lambda container image size limits](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
