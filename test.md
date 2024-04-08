# Database Implementation

## Connecting to GCP

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
