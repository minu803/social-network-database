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
  
- **Group**: Represents a group that users can join. Properties include:
  - `groupID` (string) - unique ID for the group
  - `name` (string) - name of the group
  - `description` (string) - a brief description of the group

- **Hashtag**: Represents a hashtag used in posts/comments. Properties include:
  - `name` (string) - text of the hashtag

#### Relationships

- **POSTED** (User)-[:POSTED]->(Post) - connects a User node to a Post node that the user created
- **COMMENTED_ON** (User)-[:COMMENTED_ON]->(Post) - connects a User node to a Post node that the user commented on
- **FRIEND** (User)-[:FRIEND]->(User) - connects a User node to another User node that is a friend
- **LIKED** (User)-[:LIKED]->(Post) - connects a User node to a Post node that the user liked
- **MEMBER_OF** (User)-[:MEMBER_OF]->(Group): Connects a User node to a Group node that the user is a member of.
- **TAGGED_WITH** (Post)-[:TAGGED_WITH]->(Hashtag): Connects a Post node to a Hashtag node representing a hashtag used in the post.

### Step2. Docker Setup
Using the attached `docker-compose.yml` file, first run the following command in the terminal 
``` bash
docker-compose up -d
```
and visit `http://localhost:7474` to see the database.

### Step3. Data Insertion
The example below demonstrates a segment of the nodes and relationships that were established for the project

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

  // Groups
  (groupMusic:Group {groupID: 'g1', name: 'Music Enthusiasts', description: 'A group for people who are passionate about music'}),
  (groupTravel:Group {groupID: 'g2', name: 'Travel Buffs', description: 'A group for people who love to travel and explore new places'}),
  // Additional Groups
  // ...

  // Hashtags
  (hashtagMusic:Hashtag {name: '#MusicFestival'}),
  (hashtagTravel:Hashtag {name: '#TravelTips'}),
  // Additional Hashtags
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

  // User-Group Relationships
  (josh)-[:MEMBER_OF]->(groupMusic),
  (sam)-[:MEMBER_OF]->(groupTravel),
  // Additional User-Group Relationships
  // ...

  // Post-Hashtag Relationships
  (post1)-[:TAGGED_WITH]->(hashtagMusic),
  (post2)-[:TAGGED_WITH]->(hashtagTravel),
  // Additional Post-Hashtag Relationships
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
- **Description**: Fetches all posts created by a user with a specific user ID, including details like post ID, content, timestamp, and topic.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  WHERE u.userID = '1001'
  RETURN DISTINCT u.name,p.postID, p.content, p.timestamp, p.topic
  ```
#### Find all Users who posted a specific topic of Post
- **Description**: Retrieves the names of users who have made posts on a specific topic, like 'technology'.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  WHERE p.topic = 'technology'
  RETURN DISTINCT u.name
  ```

#### Find common interests between two specific Users
- **Description**: Identifies pairs of users with shared interests and lists their common interests.
- **Example Query**:
   ```cypher
  MATCH (u1:User), (u2:User) 
  WHERE u1.userID < u2.userID AND any(interest IN u1.interests WHERE interest IN u2.interests) 
  RETURN DISTINCT u1.name, u2.name, [i IN u1.interests WHERE i IN u2.interests] AS commonInterests
  ```
#### Retrieve the top 3 Users who created the most Posts
- **Description**: Lists the top 3 users by the number of unique posts they have created, along with their names.
- **Example Query**:
  ```cypher
  MATCH (u:User)-[:POSTED]->(p:Post)
  RETURN u.name, COUNT(DISTINCT p) AS numPosts
  ORDER BY numPosts DESC
  ```
#### Retrieve Users who haven’t created any Posts
- **Description**: Finds users who have not created any posts, listing their names.
- **Example Query**:
  ```cypher
  MATCH (u:User)
  WHERE NOT EXISTS ((u)-[:POSTED]->(:Post))
  RETURN u.name
  ```
#### Identify indirect connections between two Users
- **Description**: Determines if two users are indirectly connected through friends within 2 to 3 degrees of separation and returns their names.
- **Example Query**:
  ```cypher
  MATCH path = (u1:User)-[:FRIEND*2..3]-(u3:User)
  WHERE u1 <> u3 AND length(path) > 1
  RETURN DISTINCT u1.name, u3.name
  ```
#### Identify orphaned Users (Users with no connections)
- **Description**: Identifies users who have no friends connected to them, listing their names. Useful for spotting isolated or inactive profiles.
- **Example Query**:
  ```cypher
  MATCH (u:User) 
  WHERE NOT (u)-[:FRIEND]-(:User) 
  RETURN u.name
  ```
## III. Benefits and Challenge
### Benefits
- **Enhanced Network Analysis**: Offers comprehensive insights into user interactions and content connections within the social network.
- **Targeted Content Discovery**: Advanced querying capabilities for efficiently locating specific topic-related posts by relevant users.
- **Customized User Experience**: Personalized content recommendations based on detailed analysis of user interests and network connections.

### Challenges
- **Query Complexity**: Navigating the intricacies of writing efficient graph queries, especially for complex relationships, presents a learning curve.
- **Lack of Enforced Schema**: The absence of a built-in schema can lead to inconsistencies and requires careful design to maintain data integrity.
- **Aggregation Query Limitations**: Performing certain types of aggregation queries is more complex compared to traditional SQL databases, potentially impacting data analysis efficiency.


## IV. Use of GenAI
1. **Inserting a data:**
To populate our database, I employed GenAI, which facilitated the creation of a diverse and realistic dataset. This included a range of user profiles, posts, and relational links, covering everything from basic information like names and interests to intricate inter-user connections and detailed post contents. This approach resulted in a comprehensive database that effectively simulates the dynamics of real-world social networks.

2. **Identify indirect connections between two Users:**
To identify indirect user connections within our network, I used Gen AI, which  streamlined the coding process. This approach enabled me to efficiently detect and delineate hidden social ties. A key learning aspect was learning the syntax ‘MATCH path = (u1:User)-[:FRIEND*2..3]-(u3:User)’. This pattern focuses in on FRIEND relationships, specifically targeting connections spanning 2 to 3 degrees of separation.


