# metawee-talon.one-interview-tasks

A Postman collection to test a comment functionality of a social media application.

## Task overview

Let’s pretend you work as a QA Engineer on a new social application, where users can make posts.
New functionality has just been added to allow users to create, edit and delete comments on these posts.
Your task is to create a Postman collection of tests to cover this new comment functionality, to prevent future regressions.

Please use a JSON Server to simulate the REST API responses, seeding your db.json with data from JSONPlaceholder.
This should give you a modifiable database of Posts and Comments to work with.

## Getting started

- Install [Postman](https://www.postman.com/downloads/) application
- Install JSON Server

```
npm install -g json-server
```

- Download this project
- Start JSON Server on your CLI

```
json-server --watch /{path to your db.json file}
```

- [Import the Postman collection](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/) and 'Test' environment to your Postman application
  
- [Optional] Install [Newman](https://www.npmjs.com/package/newman) to run the collection in CLI

### Run the tests

- You can run the tests individually or as a collection using [collection runner](https://learning.postman.com/docs/collections/running-collections/intro-to-collection-runs/) in Postman
- Add a small delay in the collection runner to ensure optimal results(reason: JSON Server gives random errors when the tests are running instantly after each other)
- [Run in CLI] Run

```
newman run /{path to Postman collection} -e /{path to postman environment} --delay-request 1000
```

## My testing approach

### Postman collection developing process

I started by adding requests to check on basic GET requests for related endpoints and their nested routes/filterings.

- Base Url and default post and comment ids are added to the 'Test' environment to ensure consistent results
- Tests are added to check the appropriate response status
  Next, I added POST, PUT, and PATCH requests for creating and editing comments.
- Tests are added to check the appropriate response status and body
- New post ids are saved to environment variables - so that the new comment can be created and added to the post(if run in order)
  Then I added new DELETE requests.
- Initially I used the environment variables from the previous step to set the post and comment ids for deleting but later found that the requests will be dependent on the previous requests and will fail if the requests did not run in a specific order
- So I decided to add a step to store all post and comment ids(_availablePostId_, _availableCommentId_) pre-request scripts to the GET requests where all posts and comments are called
- Then in the DELETE request I added in pre-request scripts a step to retrieve the last id from the _availablePostId_ or _availableCommentId_ array, save a new variable(_deletedPostId_, deleteCommentId) with the id to be deleted and remove it from the original array
- This is one solution to be able to run every request independently
  I duplicated the POST, PUT, and PATCH requests and altered them to test for some negative cases:
- Create a post and comment with a duplicate id
- Create a post and comment without a user id
- Update with PUT a post or comment with missing data
  After running these requests I discovered that the JSON Server API does not have validations for any cases apart from crashing with duplicated IDs so I decided to remove the requests apart from the duplicated ids cases.

Lastly, I duplicated the requests and changed them to check on the leftover routes just for the completeness of the collection.

### Testing process

After I had all Postman requests and basic regression test scripts ready I started my manual testing cases.

- Check all requests have correct responses
- Check for appropriate validations and error messages, e.g. duplicated ids, missing required request body, permission issues, etc.
- Check the interconnected relationship of resources, e.g. deleting a post also deletes its associated comments, deleting a user should also delete posts and comments

The API does not have validation or permission checks on any request that makes it possible to perform actions that would not be possible on an actual application. Nevertheless, I have listed my findings for all possible defects here:

- Creating a post or comment is possible without an id
- It is possible to send PUT requests with some missing information in the request body
- It is possible to update the id of resources
- Deleting a photo also deletes its associated album

### Improvement ideas and technical challenges

- If the application has validations and permission checks in place, I would add more negative test cases for POST, PUT, and PATCH requests to ensure proper responses.
- Currently, the most optimal way to run the collection is with the Postman collection runner because the environment variables must be updated in every run. To improve the test run in CLI or CI the collections and environment should be on the Postman cloud where they can be instantly synced. Or a new test approach has to be created.
- The collection is created with maintenance and scalability in mind. Any new request should create or use environment variables as much as possible.
- The folder structure is a draft idea now. In the future, if a final structure is decided, the duplicated tests can be moved to a folder level.
