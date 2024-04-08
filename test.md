# Database Implementation

### Connecting to GCP

`gcloud sql connect wetube --user=root;`

`show databases;`

`use youtube_database;`

`show tables;`

<img width="817" alt="Screenshot 2024-04-08 at 6 30 16 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/080a637c-39b6-4320-87ea-8775c8bb3326">
<img width="251" alt="Screenshot 2024-04-08 at 6 30 50 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/6380416b-8e41-45a8-8a8d-eb0bf807d92a">

### DDL Commands
```sql
CREATE TABLE Channel (
channelID VARCHAR(255),
channelTitle VARCHAR(255),
PRIMARY KEY(channelID)
);

CREATE TABLE Category (
categoryID INT,
categoryName VARCHAR(255),
PRIMARY KEY(categoryID)
);

CREATE TABLE User (
userID INT,
username VARCHAR(255),
password VARCHAR(255),
firstName VARCHAR(255),
lastName VARCHAR(255),
PRIMARY KEY(userID)
);

CREATE TABLE Playlist (
playlistID INT,
creatorID INT,
playlistName VARCHAR(255),
PRIMARY KEY(playlistID, creatorID),
FOREIGN KEY(creatorID) REFERENCES User(userID)
ON DELETE CASCADE
ON UPDATE CASCADE
);

CREATE TABLE Video (
videoID VARCHAR(255),
channelID VARCHAR(255),
title VARCHAR(255),
datePublished VARCHAR(255),
categoryID INT,
thumbnailLink VARCHAR(255),
PRIMARY KEY(videoID),
FOREIGN KEY(channelID) REFERENCES Channel(channelID)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY(categoryID) REFERENCES Category(categoryID)
ON DELETE CASCADE
ON UPDATE CASCADE
);

CREATE TABLE Addition (
playlistID INT,
creatorID INT,
videoID VARCHAR(255),
dateAdded VARCHAR(255),
PRIMARY KEY(playlistID, creatorID, videoID),
FOREIGN KEY(playlistID) REFERENCES Playlist(playlistID)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY(creatorID) REFERENCES Playlist(creatorID)
ON DELETE CASCADE
ON UPDATE CASCADE,
FOREIGN KEY(videoID) REFERENCES Video(videoID)
ON DELETE CASCADE
ON UPDATE CASCADE
);

CREATE TABLE VideoStats (
videoID VARCHAR(255),
trendingDate VARCHAR(255),
viewCount INT,
likes INT,
dislikes INT,
commentCount INT,
PRIMARY KEY(videoID),
FOREIGN KEY(videoID) REFERENCES Video(videoID)
ON DELETE CASCADE
ON UPDATE CASCADE
);
```
### Inserting Data (Proof of at least 1000 rows in at least 3 tables)
<img width="313" alt="Screenshot 2024-04-08 at 6 32 18 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/58876b31-c038-4c5d-9341-da64c0909549">

# Advanced Queries
### Advanced Query 1
- Advanced features: join multiple relations & aggregation via GROUP BY
- Selects the top 15 channels with the most number of videos that have a title length of 100
```sql
SELECT c.channelTitle, COUNT(v.videoID) AS num_videos
FROM Channel c JOIN Video v ON c.channelID = v.channelID
WHERE LENGTH(v.title) = 100
GROUP BY c.channelTitle
ORDER BY num_videos DESC
LIMIT 15;
```
Screenshot of the top 15 results:

<img width="510" alt="Screenshot 2024-04-08 at 6 37 23 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/500c9af3-5d5b-4a5f-a773-3d919cdcea1c">

### Advanced Query 2
- Advanced features: join multiple relations & aggregation via GROUP BY
- Selects the top 15 channels with trending videos under the most categories
```sql
SELECT ch.channelTitle, COUNT(DISTINCT ca.categoryID) AS category_count
FROM Video v JOIN Channel ch ON ch.channelID = v.channelID
JOIN Category ca ON ca.categoryID = v.categoryID
GROUP BY v.channelID
ORDER BY category_count DESC
LIMIT 15;
```
<img width="623" alt="Screenshot 2024-04-08 at 6 40 59 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/2e047cda-f519-4297-b442-20cfe2ba864b">

### Advanced Query 3
- Advanced features: join multiple relations & subquery & aggregation via GROUP BY
- For a particular category (in this screenshot, category 20 = “Gaming”), lists the top 15 channels who have posted the most videos under said category that have at least 1 million views and at most 1200 dislikes
```sql
SELECT channelTitle, videoCount
FROM (SELECT DISTINCT ch.channelTitle, COUNT(ch.channelID) AS videoCount
      FROM Video v JOIN Channel ch ON ch.channelID = v.channelID
      JOIN VideoStats vs ON v.videoID = vs.videoID
      WHERE v.categoryID = 20 AND vs.viewCount >= 1000000 AND vs.dislikes <= 1200
      GROUP BY ch.channelTitle
      HAVING videoCount > 1
      ORDER BY videoCount DESC) sub_query
GROUP BY channelTitle
LIMIT 15;
```
<img width="697" alt="Screenshot 2024-04-08 at 6 44 31 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/98f3fc28-6e8c-4539-a798-f87058ffbd70">

### Advanced Query 4
- Advanced features: join multiple relations & SET operators (UNION)
- Lists 10 videos in the “Sports” category that have at least 50000 likes and at least 1000 comments and 10 videos in the “Music” category that have at least 100000 likes and at least 5000 comments
- ```sql
  (SELECT v.title, vs.likes, vs.commentCount
   FROM Video v JOIN VideoStats vs ON v.videoID = vs.videoID
   WHERE v.categoryID = 17 AND vs.likes >= 50000 AND vs.commentCount >= 1000
   LIMIT 10)
  UNION
  (SELECT v.title, vs.likes, vs.commentCount
   FROM Video v JOIN VideoStats vs ON v.videoID = vs.videoID
   WHERE v.categoryID = 10 AND vs.likes >= 100000 AND vs.commentCount >= 5000
   LIMIT 10);
  ```
<img width="949" alt="Screenshot 2024-04-08 at 6 48 23 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/2b7758b9-84d4-4277-9d8e-c80e5c9716e0">




# Indexing Analysis
### Query 1
Analysis before indexing:
<img width="1464" alt="Screenshot 2024-04-08 at 7 08 06 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/4893e79d-d406-45e3-b9e6-347510c78ec4">



Attempt 1: `CREATE INDEX channelID_idx ON Video(channelID);`
<img width="1461" alt="Screenshot 2024-04-08 at 7 16 12 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/d00e40e2-ca9c-44ec-89e8-994a86f7964f">




Attempt 2: `CREATE INDEX title_idx ON Video(title);`
<img width="1455" alt="Screenshot 2024-04-08 at 7 20 05 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/35d7fcba-94aa-488b-ac37-1175d8f84288">





Attempt 3: `CREATE INDEX channel_title_idx ON Video(channelID, title);`
<img width="1470" alt="Screenshot 2024-04-08 at 7 21 28 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/bfad6b6b-7dd7-4561-9e3c-9c3ba53d5d79">





### Query 2
Analysis before indexing:
<img width="1460" alt="Screenshot 2024-04-08 at 7 25 15 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/041499cc-7d4d-424d-bf4d-a6f04cc921b5">




Attempt 1: `CREATE INDEX channelID_idx ON Video(channelID);`
<img width="1467" alt="Screenshot 2024-04-08 at 7 27 06 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/65d81b68-fa72-479d-bc86-6b98777665ad">





Attempt 2: `CREATE INDEX categoryID_idx ON Video(CategoryID);`
<img width="1468" alt="Screenshot 2024-04-08 at 7 29 16 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/caefbff0-b083-4503-a17e-244501e25288">







Attempt 3: `CREATE INDEX channel_category_idx ON Video(channelID, categoryID);`
<img width="1458" alt="Screenshot 2024-04-08 at 7 39 36 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/b2d7bd93-ae30-48ea-bd76-ccb84b75dbf2">




### Query 3
Analysis before indexing:
<img width="1458" alt="Screenshot 2024-04-08 at 7 45 10 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/981643c7-9450-4660-8d00-c321be7ebeee">




Attempt 1: `CREATE INDEX viewCount_idx ON VideoStats(viewCount);`
<img width="1467" alt="Screenshot 2024-04-08 at 7 47 16 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/8daf51c0-dfcd-4aa7-9f8b-e11fde8ff513">





Attempt 2: `CREATE INDEX dislikes_idx ON VideoStats(dislikes);`
<img width="1470" alt="Screenshot 2024-04-08 at 7 48 34 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/ab857933-35fc-4bb3-84c4-4681ae877eeb">







Attempt 3: `CREATE INDEX viewCount_dislikes_idx ON VideoStats(viewCount, dislikes);`
<img width="1470" alt="Screenshot 2024-04-08 at 7 50 13 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/621411b3-5dac-4fa1-9194-e4c8c647b610">






### Query 4
Analysis before indexing:
<img width="1461" alt="Screenshot 2024-04-08 at 7 53 21 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/b536711e-9800-4b38-8ffc-f92df73b3661">




Attempt 1: `CREATE INDEX likes_idx ON VideoStats(likes);`
<img width="1465" alt="Screenshot 2024-04-08 at 7 54 40 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/31be199d-5278-4bed-af6b-2fad17be002d">




Attempt 2: `CREATE INDEX commentCount_idx ON VideoStats(commentCount);`
<img width="1470" alt="Screenshot 2024-04-08 at 7 55 59 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/dd450b71-276a-4a2b-ba9c-1113e931d31c">



Attempt 3: `CREATE INDEX likes_commentCount_idx ON VideoStats(likes, commentCount);`
<img width="1469" alt="Screenshot 2024-04-08 at 7 57 14 AM" src="https://github.com/cs411-alawini/sp24-cs411-team110-wakeup/assets/109453350/72da8bfe-9547-4bdd-be0c-91eff5a75d56">

