# Social Media Graph Database

## I. Purpose
This project designs and implements a social media graph database using Neo4j. It provides practical experience with graph database modeling, Cypher querying, and integrating Neo4j with Python.

## II. Tasks
### Step1. Database Design
#### Nodes

- **User**: Represents a user on the platform. Properties include:
  - `userID` (string) - unique ID for the user
  - `name` (string) - name of the user
  - `age` (int) - age of the user
  - `location` (string) - location of the user
  - `interests` (array of strings) - interests of the user

- **Post**: Represents a post made by a user. Properties include:
  - `postID` (string) - unique ID for the post
  - `content` (string) - text content of the post
  - `timestamp` (datetime) - timestamp of when the post was created
  - `topic` (string) - topic of the post

#### Relationships

- **POSTED** (User)-[:POSTED]->(Post) - connects a User node to a Post node that the user created
- **COMMENTED_ON** (User)-[:COMMENTED_ON]->(Post) - connects a User node to a Post node that the user commented on
- **FRIEND** (User)-[:FRIEND]->(User) - connects a User node to another User node that is a friend
- **LIKED** (User)-[:LIKED]->(Post) - connects a User node to a Post node that the user liked

### Step2. Docker Setup
Using the attached `docker-compose.yml` file, first run the following command in the terminal 
``` bash
docker-compose up -d
```
and visit `http://localhost:7474` to see the database.

### Step3. Data Insertion
The example p

```cypher
CREATE
  // Users
  (josh:User {userID: '1001', name: 'Josh', age: 23, location: 'New York', interests: ['music', 'movies']}),
  (sam:User {userID: '1002', name: 'Sam', age: 25, location: 'Paris', interests: ['books', 'travel']}),
  // Additional Users
  // ...

  // Posts
  (post1:Post {postID: 'post1', content: 'Music Festival Tips', timestamp: '2023-11-10T10:00:00', topic: 'music'}),
  (post2:Post {postID: 'post2', content: 'Top Travel Destinations', timestamp: '2023-11-10T11:00:00', topic: 'travel'}),
  // Additional Posts
  // ...

  // Relationships
  (josh)-[:POSTED]->(post1),
  (josh)-[:POSTED]->(post5),
  (sam)-[:COMMENTED_ON {content: 'Interesting read.'}]->(post1),
  (riley)-[:COMMENTED_ON {content: 'I totally agree.'}]->(post3),
  (josh)-[:LIKED]->(post3),
  (sam)-[:LIKED]->(post4),
  // Additional Relationships
  // ...
```

### Step4. Data Retrieval
#### Retrieve a property of a specific User
- **Description**: Retrieves a specific property, like `name`, `age`, `location`, and `interests`  of a given user.
- **Example Query**:
  ```cypher
  MATCH (u:User {userID: '1001'}) 
  RETURN DISTINCT u.name, u.age, u.location, u.interests
  ```
#### Find all Posts created by a specific User
- **Description**: Lists all the posts created by a particular user.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  WHERE u.userID = '1001'
  RETURN DISTINCT u.name,p.postID, p.content, p.timestamp, p.topic
  ```
#### Find all Users who posted a specific topic of Post
- **Description**: Identifies users who have posted about a specific topic.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  WHERE p.topic = 'technology'
  RETURN DISTINCT u.name
  ```

#### Find common interests between two specific Users
- **Description**: Finds the shared interests between two given users.
- **Example Query**:
   ```cypher
  MATCH (u1:User), (u2:User) 
  WHERE u1.userID < u2.userID AND any(interest IN u1.interests WHERE interest IN u2.interests) 
  RETURN DISTINCT u1.name, u2.name, [i IN u1.interests WHERE i IN u2.interests] AS commonInterests
  ```
#### Retrieve the top 3 Users who created the most Posts
- **Description**: Lists the top 3 users who have created the most posts.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  RETURN u.name, COUNT(DISTINCT p) AS numPosts
  ORDER BY numPosts DESC
  ```
#### Retrieve Users who haven’t created any Posts
- **Description**: Identifies users who have not created any posts.
- **Example Query**:
  ```cypher
  MATCH (u:User)
  WHERE NOT EXISTS ((u)-[:POSTED]->(:Post))
  RETURN u.name
  ```
#### Identify indirect connections between two Users
- **Description**: : Determines if two users are indirectly connected through a chain of friends and returns the the names.
- **Example Query**:
  ```cypher
  MATCH path = (u1:User)-[:FRIEND*2..3]-(u3:User)
  WHERE u1 <> u3 AND length(path) > 1
  RETURN DISTINCT u1.name, u3.name
  ```
#### Identify orphaned Users (Users with no connections)
- **Description**: Find users who have no connections (friends, posts, etc.). Useful for identifying inactive or isolated users.
- **Example Query**:
  ```cypher
  MATCH (u:User) 
  WHERE NOT (u)-[:FRIEND]-(:User) 
  RETURN u.name
  ```
## III. Use of GenAI



