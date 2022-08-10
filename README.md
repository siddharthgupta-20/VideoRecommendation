# VideoRecommendation


DATA COLLECTION:
APIs Used -> Youtube Api v3
Filters Used -> Region code : IN to get data relevant to this region only, Relevance Lang : English
I have collected data for 3 days. On the first day, I queried my topic and took a total of 1500 data points
for all topics together (500 per topic). On day 6, I queried again 1500 data points using the same function
as before. On day 7, I took 1500 data points containing exactly the same videos as on day 1 (this helped
me capture the trend).
Note: In day 6, there are new videos too which were not on day 1 but day 1 and day 7 have same videos.
I took data in this way to check the change in how the api is returning data, i.e., to capture how youtube
data is being ranked (I used this later in my scoring function)
I have also added a column in the dataset for commentCount to capture activity of a video (However I did
not use this column).
I have attached both original_data.csv which does not contain score and 11841160_data.csv which
contains values of the score. Both have unnormalized data.

METHODOLOGY:
After collecting data, I have done some cleaning such as removing certain duplicates, getting rid of entries
with inconsistent data.There were some entries for which likeCount, dislikeCount were missing, for these I
replaced them with the mean of that column. Then I normalized viewCount, likeCount and dislikeCount
using z score normalization.
Then I have computed the score of the video using:
score = 3*float(likeCount)+0.5*float(viewCount)-1.5*float(dislikeCount)-((3-freq)*freq*freq)/3 +
0.5*trend)
There are two new terms in this , namely, ((3-freq)*freq^2)/3 and 0.5*trend.
The explanation of these terms is as follows:

● ((3-freq)*freq^2)/3 :- Here freq refers to the frequency of that video_id in my dataset.
Since I have collected data on 3 days as explained above, it may be possible that one video may
occur on day1, day6, day7 or it will occur only on day6 or it will occur on both day1 and day7.
This can be expected (I have explained in data collection why this happens) and frequency of
video_id will be either 3 or 1 or 2 respectively.
I have introduced this term to capture the rankings of how youtube api returns data also. It is
possible that a particular video is on day 1 will not be on day 6, i.e. ,it is not in top(500) rankings
of youtube of that day which I got from API. In that case this term will give a penalty to that data
because if it is not in best youtube results on a particular day then it means that the video is not
that good. Freq = 2 and more penalty (based on term)
If the video is present on day 1 and day6 then its frequency will become 3 (Since if it is present in
day 1, it will automatically appear on day 7 by method in which i collected) and hence in this case
penalty will become 0 as should be the case as video must be a good video. Freq = 3 and hence
term equates to 0 (no penalty)
If video is obtained on day 6 but not on day 1, then it must be that the video might be a new one
and is getting better if it is being upgraded in youtube rankings also, so penalty for this will be
lesser than if it were present on day 1 but not day 6 ( It is getting little penalty because it is not on
day 1). Freq = 1 and hence less penalty.
Keeping in mind the above three possibilities, I made this term as it is to give penalty depending
on the above three cases.

● Trend :
This term is defined as 2*(dif(likeCount))+(dif(viewCount))-1.5*(dif(dislikeCount))
Where dif(x) is the difference of x between day 1 and day 7 as these contain the same videos.
For those videos occurring on day 1 but not on day 6 and those occurring on day 6 but not day 1,
I took trend to be the mean of other trends.
This term as it is named captures how the video statistics are changing and can be used in the
scoring function.
After computing the scoring function, I ranked the unique videos based on their scores, i.e., higher score
means higher rank.
For generating the playlist, I used the following method,:
I kept putting the videos in the playlist, starting from the video with the highest rank and going downwards
as long as the total time of playlist remained less than 15 hours. So the playlist contains the best ranked
videos (Similar to a greedy algorithm).

OBSERVATIONS:
I have analyzed the average of likeCount, viewCount and dislikeCount for different video
duration lengths. I found that generally longer videos have higher likes and relatively more
views.
