---
layout: page
title: Running on AWS Lambda
nav_order: 800
parent: Backend
---

# Running the backend on AWS Lambda

This page contains various notes on the deployment of the backend service on AWS Lambda.

It is interesting to understand how lambdas work under the hood. In nutshell: a daemon makes a call on an [API endpoint](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html) (`/next`) which is blocking waiting for the next (HTTP) request to arrive. Then it passes the request to your lambda code with a timeout, and send the response back to that endpoint, and then wait back in loop on this `/next` endpoint. You do not know how many requests the daemon will process before being stopped and started on another server.


## Running code after returning a response from an AWS Lambda function

This may be useful to run some cleanup or some processing while responding quite quickly. The typical use case is propagations.

[This interesting blog post from AWS](https://aws.amazon.com/blogs/compute/running-code-after-returning-a-response-from-an-aws-lambda-function/) describes several ways to achieve this goal.

About the different options:
- Invoking an async worker is a valid solution (used for propagation on June 2024) but still suffer to the delay to the async call. This is not really negligible, especially if we can encounter a cold start.

- Using Go compiled code, we are already using a custom runtime. We could probably fork the lambda lib, and change the processing loop to call a post-processing API endpoint on our app between each actual call. 

- *Response steaming* looks like a great way to achieve the same goal but for now, it is unclear how it can be used with standalone (OS level runtime) provider and Golang. As it is quite new, documentations and lib updates may still appear.

- Internal extensions look like an option as well, but currently no examples using Go running code after returning have been found. See [doc](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-extensions-api.html) and [examples](https://github.com/aws-samples/aws-lambda-extensions/tree/main).

## Small improvements that should be done soon (on June 2024)

- the `algnhsa` lib that we use is completely useless with the standalone provider that we (must) run now, actually it just call the official AWS lambda Go lib with:
```
lambdaHandler := New(handler)
lambda.StartHandler(lambdaHandler)
```

- we could use the `al2023` provider instead of `al2`

- we could compile without linking to libc as explained [here](https://github.com/aws/aws-lambda-go?tab=readme-ov-file#for-developers-on-linux)

