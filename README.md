# Evaluate Performance at Massive Scale workshop

- [Evaluate Performance at Massive Scale workshop](#evaluate-performance-at-massive-scale-workshop)
  - [(1) Workshop Intro](#1-workshop-intro)
    - [(1.1) Introduction](#11-introduction)
    - [(1.2) Prerequisites](#12-prerequisites)
    - [(1.3) Playground](#13-playground)
  - [(2) Foundations](#2-foundations)
    - [(2.1) First test](#21-first-test)
    - [(2.2) VUs and Iterations](#22-vus-and-iterations)


## (1) Workshop Intro

### (1.1) Introduction

Performance testing is the practice that helps us verify the reliability and performance of our applications.

Reliability has become paramount, and it is now a shared responsibility across all teams. Thus, performance testing has also shifted left, moving closer to frontend and backend developers.

Grafana k6 is a modern, open-source performance testing tool scriptable in Javascript. This interactive workshop provides participants with hands-on experience writing k6 performance tests, giving an overview of performance testing practices and various options for using k6 through practical examples with a demo app.Participants will use Docker compose to install locally a demo environment. 

**Objectives:**

- Learn performance testing fundamentals and guidelines

- Implement different type of load tests and browser performance tests

- Model load tests in terms of throughput and concurrent users

- Implement advanced test scenarios

- Visualize performance results with OSS stack

- Verify SLO compliance in performance tests

- Extend k6 using extensions

### (1.2) Prerequisites

We need requirements to follow the hands-on lab today

* Docker Engine
* VS Code
* Optional: k6
  * You can run it inside Docker, but the experience is better if you install it locally.
    * You get nice colors and dynamic progress bars!
    * Also, it is just a binary, so you can easily remove it afterward if you don't want it.
  * If you plan to use Docker, please, pre-pull the images with:
      ```
      docker pull grafana/k6:0.47.0
      docker pull grafana/k6:0.47.0-with-browser
      ```

### (1.3) Playground

First of all, clone the repository:
```
git clone git@github.com:Athens-SDET-Meetup/devoxx_workshop.git
```
Then, run the playground with:
```
cd devoxx_workshop; docker compose up -d
```
To verify everything is working, go to http://localhost:3333 and click the big button.
If you see a pizza recommendation, that's good!
Also, open http://localhost:3000 and verify that you can see a Grafana instance.

## (2) Foundations

### (2.1) First test

To run your first test just copy the following

import http from "k6/http";

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3333';

```
export default function () {
  let restrictions = {
    maxCaloriesPerSlice: 500,
    mustBeVegetarian: false,
    excludedIngredients: ["pepperoni"],
    excludedTools: ["knife"],
    maxNumberOfToppings: 6,
    minNumberOfToppings: 2
  }
  let res = http.post(`${BASE_URL}/api/pizza`, JSON.stringify(restrictions), {
    headers: {
      'Content-Type': 'application/json',
      'X-User-ID': 23423,
    },
  });
  console.log(`${res.json().pizza.name} (${res.json().pizza.ingredients.length} ingredients)`);
}
```
in a file called example.js.

Then, run it with:
```
# If you have k6 installed
k6 run example.js
# If you don't have k6 installed
docker run -i --network=devfest-santiago_default grafana/k6:0.47.0 run -e BASE_URL=http://quickpizza:3333  - <example.js
```
That's it :muscle:! You have successfully run your first test :grinning:

Now, you have a lot of things in the output. Let's go through them.

k6's output has three main sections:

k6's output has three main sections:
- 1. Information about the test and its configuration.
  - Why: To quickly understand the test configuration and how it will run.
- 2. Runtime information (e.g., logs).
  - Why: To understand what's happening during the test.
- 3. Summary of the test.
  - Why: To understand how the test went.

In the output's second section, you should see a log line with the pizza name and the number of ingredients. This happens once because k6 has run your default function once. You can also see in the third section lots of metrics that k6 has generated and aggregated for you. These metrics are helpful to understand how the test went (e.g., the number of requests, errors, response time, etc.).


### (2.2) VUs and Iterations

In k6, we have two main concepts: Virtual Users (VUs) and iterations. A VU is a virtual user. It's a thread that runs your script. It's the basic unit of execution in k6. An iteration is a single execution of your script. In the example above, it's a single run of your default function.

You can think of it like a for-loop. You can have many of those; each one will run your script. The number of iterations is the number of times that your script will run. The number of VUs is the number of threads that will run your script.

![vus and iterations](./media/vus.png)

Let's try it out! After the imports, add a configuration block to your script:
```javascript
export let options = {
  vus: 10,
  duration: "5s",
};
```

Then, run it again.

You should see many more log lines with pizza recommendations in the output. Many things change in the output when you increase the number of VUs and duration, like the values of the VUs and iterations metrics, the number of requests, the response time, etc.

