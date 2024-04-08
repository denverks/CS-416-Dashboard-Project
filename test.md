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

SELECT channelID, channelTitle, videoCount
FROM (
    SELECT DISTINCT ch.channelID, ch.channelTitle, COUNT(ch.channelID) AS videoCount
    FROM Video v
    JOIN Channel ch ON ch.channelID = v.channelID
    JOIN VideoStats vs ON v.videoID = vs.videoID
    WHERE v.categoryID = 17
    AND vs.viewCount >= 1000000
    AND vs.dislikes <= 100
    GROUP BY ch.channelID, ch.channelTitle
    HAVING videoCount > 1
    ORDER BY videoCount DESC
) t1
GROUP BY channelID, channelTitle
LIMIT 15;
```
